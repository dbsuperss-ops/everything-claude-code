---
description: 프로젝트를 자동 분석하여 PM2(Process Manager 2) 서비스 관리 설정 및 명령어를 생성합니다.
---

# PM2 초기화 (PM2 Init)

프로젝트 구조를 분석하여 백엔드, 프론트엔드 등의 서비스를 PM2로 관리할 수 있도록 설정 파일과 명령어를 생성합니다.

## 워크플로우

1. **PM2 확인**: 시스템에 PM2가 설치되어 있는지 확인하고, 없는 경우 설치를 안내하거나 진행합니다.
2. **서비스 감지**: 프로젝트 내의 프론트엔드(Vite, Next.js 등), 백엔드(Node.js, Python, Go 등) 서비스를 자동으로 찾아냅니다.
3. **설정 생성**: `ecosystem.config.cjs` 파일과 개별 서비스 제어용 명령어를 생성합니다.

---

## 서비스 감지 기준

| 유형 | 감지 방식 | 기본 포트 |
|------|-----------|--------------|
| Vite | `vite.config.*` 존재 여부 | 5173 |
| Next.js | `next.config.*` 존재 여부 | 3000 |
| Express/Node | `package.json` 및 서버 디렉토리 분석 | 3000 |
| Fast API/Flask | `requirements.txt` / `pyproject.toml` 분석 | 8000 |
| Go | `go.mod` / `main.go` 분석 | 8080 |

**포트 결정 우선순위**: 사용자 지정 > `.env` 파일 > 설정 파일 > 기본 포트

---

## 생성되는 파일 구조

```text
프로젝트 루트/
├── ecosystem.config.cjs              # PM2 메인 설정 파일
├── {backend}/start.cjs               # Python 등을 위한 래퍼 (필요시)
└── .claude/
    ├── commands/
    │   ├── pm2-all.md                # 전체 시작 및 모니터링
    │   ├── pm2-all-stop.md           # 전체 중지
    │   ├── pm2-all-restart.md        # 전체 재시작
    │   ├── pm2-{포트}.md             # 단일 서비스 시작 및 로그 확인
    │   ├── pm2-logs.md               # 모든 로그 보기
    │   └── pm2-status.md             # 프로세스 상태 확인
    └── scripts/
        ├── pm2-logs-{포트}.ps1       # 단일 서비스 로그 스크립트
        └── pm2-monit.ps1             # 모니터링 패널 스크립트
```

---

## Windows 환경 설정 (중요)

Windows에서는 호환성을 위해 **`.cjs` 확장자**를 반드시 사용해야 합니다.

### ecosystem.config.cjs 예시
```javascript
module.exports = {
  apps: [
    {
      name: 'web-app-3000',
      cwd: './frontend',
      script: 'node_modules/vite/bin/vite.js',
      args: '--port 3000',
      interpreter: 'node',
      env: { NODE_ENV: 'development' }
    },
    {
      name: 'api-server-8000',
      cwd: './backend',
      script: 'start.cjs', // Python 래퍼
      env: { PYTHONUNBUFFERED: '1' }
    }
  ]
}
```

---

## 주요 명령어 사용법

### 초기 실행 (설정 파일 적용)
```bash
pm2 start ecosystem.config.cjs && pm2 save
```

### 상시 관리 (저장된 프로세스 리스트 활용)
* **전체 시작**: `pm2 start all`
* **전체 중지**: `pm2 stop all`
* **로그 확인**: `pm2 logs`
* **모니터링 패널**: `pm2 monit`
* **상태 확인**: `pm2 status`
* **리스트 저장**: `pm2 save` (재부팅 후 복구용)

---

## 초기화 완료 후 요약 보고

모든 설정이 완료되면 에이전트는 다음과 같은 요약을 제공합니다:

```text
## PM2 초기화 완료 리포트

**감지된 서비스:**
| 포트 | 이름 | 유형 |
|------|------|------|
| 3000 | my-frontend | Vite |
| 8000 | my-api | FastAPI |

**사용 가능한 명령어:**
/pm2-all, /pm2-all-stop, /pm2-logs, /pm2-status 등

**팁:** 첫 실행 후 `pm2 save`를 실행하면 컴퓨터 재부팅 시에도 `pm2 resurrect` 명령오로 서비스를 즉시 복구할 수 있습니다.
```

**핵심**: PM2 초기화 명령어는 개발 환경의 수많은 프로세스를 체계적으로 관리하고, 로그 확인 및 모니터링을 한 곳에서 수행할 수 있도록 돕습니다.
