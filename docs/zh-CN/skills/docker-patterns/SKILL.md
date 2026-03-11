---
name: docker-patterns
description: 로컬 개발을 위한 Docker 및 Docker Compose 패턴입니다. 컨테이너 보안, 네트워크, 볼륨 전략 및 다중 서비스 오케스트레이션을 다룹니다.
origin: ECC
---

# Docker 패턴

컨테이너 기반 개발을 위한 Docker 및 Docker Compose 베스트 프랙티스입니다.

## 적용 시점

* 로컬 개발 환경 구축을 위해 Docker Compose를 설정할 때
* 다중 컨테이너(Multi-container) 아키텍처를 설계할 때
* 컨테이너 네트워크 또는 볼륨 관련 문제를 해결(Troubleshooting)할 때
* Dockerfile의 보안성 및 이미지 크기를 최적화하고 리뷰할 때
* 로컬 개발 환경에서 컨테이너 기반 워크플로우로 이관할 때

## 로컬 개발용 Docker Compose

### 표준 웹 애플리케이션 스택 구성 예시

```yaml
# docker-compose.yml
services:
  app:
    build:
      context: .
      target: dev                     # 멀티 스테이지 Dockerfile의 dev 스테이지 사용
    ports:
      - "3000:3000"
    volumes:
      - .:/app                        # 소스 코드 핫 리로드(Hot reload)를 위한 바인드 마운트
      - /app/node_modules             # 익명 볼륨 - 컨테이너 내부의 의존성 보존
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/app_dev
      - REDIS_URL=redis://redis:6379/0
      - NODE_ENV=development
    depends_on:
      db:
        condition: service_healthy    # DB가 헬스체크를 통과한 후 실행
      redis:
        condition: service_started
    command: npm run dev

  db:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app_dev
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data

  mailpit:                            # 로컬 이메일 테스트 도구
    image: axllent/mailpit
    ports:
      - "8025:8025"                   # 웹 UI
      - "1025:1025"                   # SMTP 호스트

volumes:
  pgdata:
  redisdata:
```

### 개발(Dev) 및 운영(Prod) 공용 Dockerfile

```dockerfile
# 1단계: 의존성 설치 (deps)
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# 2단계: 개발 환경 (dev - 핫 리로드, 디버그 도구 포함)
FROM node:22-alpine AS dev
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# 3단계: 빌드 환경 (build)
FROM node:22-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build && npm prune --production

# 4단계: 운영 환경 (production - 최소화된 이미지)
FROM node:22-alpine AS production
WORKDIR /app
RUN addgroup -g 1001 -S appgroup && adduser -S appuser -u 1001
USER appuser
COPY --from=build --chown=appuser:appgroup /app/dist ./dist
COPY --from=build --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=build --chown=appuser:appgroup /app/package.json ./
ENV NODE_ENV=production
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

### 오버라이드(Override) 파일 활용

```yaml
# docker-compose.override.yml (자동 로드됨, 개발 전용 설정)
services:
  app:
    environment:
      - DEBUG=app:*
      - LOG_LEVEL=debug
    ports:
      - "9229:9229"                   # Node.js 디버거 포트

# docker-compose.prod.yml (운영 환경 전용 명시적 설정)
services:
  app:
    build:
      target: production
    restart: always
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
```

```bash
# 개발 환경 (override 파일 자동 포함)
docker compose up

# 운영 환경 (명시적으로 파일 지정)
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## 네트워크 (Networking)

### 서비스 검색 (Service Discovery)
동일한 Compose 네트워크 내부에서는 서비스 이름을 통해 통신이 가능합니다:

```
# "app" 컨테이너 내부에서 접근 시:
postgres://postgres:postgres@db:5432/app_dev    # "db"가 DB 컨테이너 IP로 해석됨
redis://redis:6379/0                             # "redis"가 Redis 컨테이너 IP로 해석됨
```

### 사용자 정의 네트워크 (Isolation)

```yaml
services:
  frontend:
    networks:
      - frontend-net

  api:
    networks:
      - frontend-net
      - backend-net

  db:
    networks:
      - backend-net              # API 컨테이너에서만 접근 가능, 프론트엔드에서는 접근 불가

networks:
  frontend-net:
  backend-net:
```

## 볼륨 전략 (Volumes)

```yaml
volumes:
  # 이름 있는 볼륨 (Named volume): 컨테이너 재시작 후에도 유지되며 Docker가 관리함
  pgdata:

  # 바인드 마운트 (Bind mount): 호스트의 디렉토리를 컨테이너에 매핑함 (개발용 코드 수정 반영)
  # - ./src:/app/src

  # 익명 볼륨 (Anonymous volume): 바인드 마운트가 컨테이너 내부의 특정 폴더를 덮어쓰지 않도록 보호함
  # - /app/node_modules
```

## 컨테이너 보안

### Dockerfile 보안 강화 규칙
1. **특정 버전 태그 사용**: `:latest`는 절대 사용하지 마십시오 (예: `node:22.12-alpine3.20`).
2. **Non-root 계정 실행**: root가 아닌 전용 사용자를 생성하고 `USER` 명령으로 적용하십시오.
3. **Capabilities 제한**: Compose 설정에서 불필요한 권한(`cap_drop`)을 제거하십시오.
4. **읽기 전용 파일시스템**: 가능한 경우 `read_only: true` 설정을 적용하십시오.
5. **이미지 레이어에 비밀 정보 포함 금지**: 빌드 시점에 API Key 등을 박아넣지 마십시오.

### 시크릿 및 환경 변수 관리
* **권장**: 런타임에 주입되는 `.env` 파일(Git 제외 필수)이나 호스트 환경 변수를 사용하십시오.
* **권장**: Docker 컨테이너 오케스트레이션 도구(Swarm, K8s)의 Secret 기능을 활용하십시오.
* **금지**: Dockerfile 내부에 `ENV API_KEY=xxxx`와 같이 하드코딩하지 마십시오.

## .dockerignore 설정 예시
```
node_modules
.git
.env
.env.*
dist
coverage
*.log
.next
.cache
docker-compose*.yml
Dockerfile*
README.md
tests/
```

## 디버깅 및 관리

### 주요 명령어
* **로그 확인**: `docker compose logs -f [서비스명]`
* **컨테이너 진입**: `docker compose exec [서비스명] sh`
* **상태 확인**: `docker compose ps`, `docker stats` (리소스 사용량)
* **이미지 빌드**: `docker compose up --build`, `docker compose build --no-cache`
* **정리**: `docker compose down -v` (볼륨까지 삭제 - 주의), `docker system prune`

### 네트워크 트러블슈팅
* **DNS 확인**: `docker compose exec app nslookup db`
* **연결 확인**: `docker compose exec app wget -qO- http://api:3000/health`

## 피해야 할 안티 패턴

* **운영 환경에서 단순 Compose 사용**: 운영 환경에서는 Kubernetes나 ECS 같은 오케스트레이션 도구를 권장합니다.
* **볼륨 없이 데이터 저장**: 컨테이너는 일시적(Ephemeral)입니다. 볼륨 없이는 데이터가 유실됩니다.
* **중복 계정(root) 사용**: 보안을 위해 항상 일반 사용자 계정을 사용하십시오.
* **이미지 하나에 모든 서비스 구동**: '컨테이너당 하나의 프로세스' 원칙을 준수하여 서비스를 분리하십시오.
* **시크릿 정보 노출**: `docker-compose.yml` 파일에 비밀번호를 직접 적지 마십시오.
