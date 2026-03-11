---
name: deployment-patterns
description: 배포 워크플로우, CI/CD 파이프라인 패턴, Docker 컨테이너화, 상태 확인(Health check), 롤백 전략 및 웹 애플리케이션의 운영 환경 준비(Production readiness) 체크리스트입니다.
origin: ECC
---

# 배포 패턴

운영 환경 배포 워크플로우 및 CI/CD 베스트 프랙티스입니다.

## 적용 시점

* CI/CD 파이프라인을 설정할 때
* 애플리케이션을 컨테이너화(Docker)할 때
* 배포 전략(Blue-Green, Canary, Rolling 등)을 계획할 때
* 상태 확인(Health check) 및 준비성 프로브(Readiness probe)를 구현할 때
* 운영 환경 출시를 준비할 때
* 환경별 설정을 구성할 때

## 배포 전략

### 롤링 배포 (Rolling Deployment - 기본값)
인스턴스를 점진적으로 교체합니다. 배포 프로세스 동안 이전 버전과 새 버전이 동시에 실행됩니다.

```
인스턴스 1: v1 → v2  (첫 번째 업데이트)
인스턴스 2: v1        (여전히 v1 실행 중)
인스턴스 3: v1        (여전히 v1 실행 중)

인스턴스 1: v2
인스턴스 2: v1 → v2  (두 번째 업데이트)
인스턴스 3: v1

인스턴스 1: v2
인스턴스 2: v2
인스턴스 3: v1 → v2  (마지막 업데이트)
```

**장점:** 가동 중단 시간 없음(Zero downtime), 점진적 출시
**단점:** 두 버전이 동시에 공존하므로 하위 호환성(Backward compatibility) 보장이 필수적임
**적용:** 표준 배포, 호환성이 유지되는 변경 사항

### 블루-그린 배포 (Blue-Green Deployment)
동일한 두 환경을 운영합니다. 트래픽을 원자적(Atomic)으로 전환합니다.

```
블루 (v1) ← 트래픽 유입
그린 (v2)   대기 중, 새 버전 실행 중

# 검증 완료 후:
블루 (v1)   대기 중 (백업으로 전환)
그린 (v2) ← 트래픽 유입
```

**장점:** 즉각적인 롤백 가능(블루로 다시 전환), 깔끔한 전환
**단점:** 배포 기간 동안 인프라 자원이 2배로 필요함
**적용:** 주요 서비스, 장애 무관용 환경

### 카나리 배포 (Canary Deployment)
전체 트래픽의 아주 적은 일부만 먼저 새 버전으로 보냅니다.

```
v1: 트래픽의 95%
v2: 트래픽의  5%  (카나리 서버)

# 지표가 안정적이면:
v1: 트래픽의 50%
v2: 트래픽의 50%

# 최종 완료:
v2: 트래픽의 100%
```

**장점:** 실제 트래픽을 통해 전체 출시 전 문제를 조기에 발견할 수 있음
**단점:** 트래픽 분산 인프라 및 정교한 모니터링이 필요함
**적용:** 트래픽이 많은 서비스, 위험 부담이 큰 변경, 기능 플래그 활용 시

## Docker

### 멀티 스테이지 Dockerfile (Node.js)

```dockerfile
# 1단계: 의존성 설치
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --production=false

# 2단계: 빌드
FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build
RUN npm prune --production

# 3단계: 실행 이미지
FROM node:22-alpine AS runner
WORKDIR /app

RUN addgroup -g 1001 -S appgroup && adduser -S appuser -u 1001
USER appuser

COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/package.json ./

ENV NODE_ENV=production
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

### 멀티 스테이지 Dockerfile (Go)

```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /server ./cmd/server

FROM alpine:3.19 AS runner
RUN apk --no-cache add ca-certificates
RUN adduser -D -u 1001 appuser
USER appuser

COPY --from=builder /server /server

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:8080/health || exit 1
CMD ["/server"]
```

### 멀티 스테이지 Dockerfile (Python/Django)

```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
RUN pip install --no-cache-dir uv
COPY requirements.txt .
RUN uv pip install --system --no-cache -r requirements.txt

FROM python:3.12-slim AS runner
WORKDIR /app

RUN useradd -r -u 1001 appuser
USER appuser

COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY . .

ENV PYTHONUNBUFFERED=1
EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health/')" || exit 1
CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

### Docker 베스트 프랙티스

```
# 권장 사항 (DO)
- 특정 버전 태그를 사용하십시오 (node:latest 대신 node:22-alpine 등)
- 이미지 크기 최소화를 위해 멀티 스테이지 빌드를 적용하십시오
- root가 아닌 계정(non-root user)으로 실행하십시오
- 의존성 파일을 먼저 복사하여 레이어 캐싱을 활용하십시오
- .dockerignore를 사용하여 node_modules, .git, 테스트 코드를 제외하십시오
- HEALTHCHECK 명령을 추가하십시오
- docker-compose나 k8s에서 리소스 제한(Limit)을 설정하십시오

# 금지 사항 (DON'T)
- root 권한으로 실행하지 마십시오
- :latest 태그를 사용하지 마십시오
- 하나의 COPY 레이어에서 전체 저장소를 복사하지 마십시오
- 운영 이미지에 개발용 의존성(dev dependencies)을 포함하지 마십시오
- 이미지 내부에 비밀 정보(Secret)를 저장하지 마십시오 (환경 변수나 보안 관리 도구 사용)
```

## CI/CD 파이프라인

### GitHub Actions (표준 파이프라인)

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage
          path: coverage/

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - name: 운영 환경 배포
        run: |
          # 플랫폼별 배포 명령 실행 예시
          # Railway: railway up
          # Vercel: vercel --prod
          # K8s: kubectl set image deployment/app app=ghcr.io/${{ github.repository }}:${{ github.sha }}
          echo "배포 버전: ${{ github.sha }}"
```

### 파이프라인 단계 구성 예시

```
PR 생성 시:
  lint → typecheck → 단위 테스트 → 통합 테스트 → 프리뷰 배포

Main 병합 시:
  lint → typecheck → 단위 테스트 → 통합 테스트 → 이미지 빌드 → 스테이징 배포 → 스모크 테스트 → 운영 배포
```

## 상태 확인 (Health Checks)

### 상태 확인 엔드포인트 구현

```typescript
// 단순 상태 확인
app.get("/health", (req, res) => {
  res.status(200).json({ status: "ok" });
});

// 상세 상태 확인 (내부 모니터링용)
app.get("/health/detailed", async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    externalApi: await checkExternalApi(),
  };

  const allHealthy = Object.values(checks).every(c => c.status === "ok");

  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? "ok" : "degraded",
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION || "unknown",
    uptime: process.uptime(),
    checks,
  });
});

async function checkDatabase(): Promise<HealthCheck> {
  try {
    await db.query("SELECT 1");
    return { status: "ok", latency_ms: 2 };
  } catch (err) {
    return { status: "error", message: "데이터베이스 연결 실패" };
  }
}
```

### Kubernetes 프로브(Probes) 설정

```yaml
livenessProbe: # 앱이 살아있는지 확인 (실패 시 재시작)
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 30
  failureThreshold: 3

readinessProbe: # 앱이 요청을 받을 준비가 되었는지 확인 (실패 시 트래픽 차단)
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 10
  failureThreshold: 2

startupProbe: # 앱이 완전히 기동되었는지 확인 (대규모 앱 초기화 대기용)
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 0
  periodSeconds: 5
  failureThreshold: 30    # 30 * 5초 = 최대 150초까지 대기
```

## 환경 설정

### Twelve-Factor App 패턴
```bash
# 모든 설정은 환경 변수를 통해 관리하십시오 (절대 코드에 하드코딩하지 마십시오)
DATABASE_URL=postgres://user:pass@host:5432/db
REDIS_URL=redis://host:6379/0
API_KEY=${API_KEY}           # 보안 관리 도구에서 주입
LOG_LEVEL=info
PORT=3000

# 환경 구분
NODE_ENV=production          # 또는 staging, development
APP_ENV=production           # 명시적인 앱 실행 환경
```

### 설정 검증 (Config Validation)

```typescript
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z.enum(["development", "staging", "production"]),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  LOG_LEVEL: z.enum(["debug", "info", "warn", "error"]).default("info"),
});

// 기동 시점에 즉시 검증하여 설정 오류가 있으면 즉시 실패 처리(Fail Fast)
export const env = envSchema.parse(process.env);
```

## 롤백 전략

### 즉시 롤백 명령 예시
```bash
# Docker/Kubernetes: 이전 이미지로 복구
kubectl rollout undo deployment/app

# Vercel: 이전 배포본으로 승격
vercel rollback

# Railway: 이전 커밋으로 재배포
railway up --commit <이전-SHA>

# 데이터베이스: 마이그레이션 롤백 (가역적인 경우)
npx prisma migrate resolve --rolled-back <마이그레이션-이름>
```

### 롤백 필수 체크리스트
* [ ] 이전 버전의 이미지/산출물이 즉시 사용 가능한 상태인가.
* [ ] 데이터베이스 마이그레이션이 하위 호환성을 유지하는가(삭제 작업 분리).
* [ ] 기능 플래그(Feature Flags)를 통해 배포 없이 새 기능을 비활성화할 수 있는가.
* [ ] 에러율 급증 시 알림이 오도록 모니터링이 설정되어 있는가.
* [ ] 운영 환경 배포 전 스테이징 환경에서 롤백 테스트를 완료했는가.

## 운영 환경 준비(Production Readiness) 체크리스트

배포 전 최종 확인 사항:

### 애플리케이션
* [ ] 모든 테스트 통과 (단위, 통합, E2E).
* [ ] 코드나 설정 파일에 하드코딩된 비밀 정보가 없는가.
* [ ] 모든 엣지 케이스에 대한 에러 처리가 되어 있는가.
* [ ] 로그가 구조화(JSON)되어 있으며 개인정보(PII)를 포함하지 않는가.
* [ ] 상태 확인 엔드포인트가 의미 있는 상태를 반환하는가.

### 인프라스트럭처
* [ ] Docker 이미지가 재현 가능한 방식으로 빌드되었는가 (버전 고정).
* [ ] 환경 변수가 문서화되어 있고 기동 시점에 검증되는가.
* [ ] 리소스 한도(CPU, Memory)가 설정되어 있는가.
* [ ] 수평적 확장(Auto-scaling)이 설정되어 있는가.
* [ ] 모든 엔드포인트에 SSL/TLS가 적용되었는가.

### 모니터링
* [ ] 핵심 지표(요청률, 지연 시간, 에러율)가 수집되고 있는가.
* [ ] 임계값 초과 시 알림(Alerting)이 설정되어 있는가.
* [ ] 로그 수집 및 검색 시스템이 구축되어 있는가.
* [ ] 업타임 모니터링이 가동 중인가.

### 보안
* [ ] 의존성 라이브러리의 취약점(CVE) 스캔을 통과했는가.
* [ ] CORS 설정이 허용된 출처로만 제한되어 있는가.
* [ ] 공개 엔드포인트에 속도 제한(Rate limiting)이 적용되어 있는가.
* [ ] 인증 및 인가 로직이 검증되었는가.
* [ ] 보안 헤더(CSP, HSTS, X-Frame-Options 등)가 설정되었는가.

### 운영 및 장애 대응
* [ ] 롤백 계획이 문서화되고 테스트되었는가.
* [ ] 데이터베이스 마이그레이션이 대규모 데이터에서 테스트되었는가.
* [ ] 주요 장애 시나리오에 대한 매뉴얼이 있는가.
* [ ] 온콜(On-call) 당번 및 비상 연락망이 정의되어 있는가.
