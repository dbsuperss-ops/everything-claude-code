---
description: 선호하는 패키지 매니저(npm/pnpm/yarn/bun)를 프로젝트 또는 전역 단위로 설정합니다.
---

# 패키지 매니저 설정 (Setup-PM)

현재 프로젝트 또는 전체 시스템에서 사용할 패키지 매니저를 선택하고 관리합니다.

## 사용법

```bash
# 현재 프로젝트의 패키지 매니저 감지
node scripts/setup-package-manager.js --detect

# 전역(Global) 선호 매니저 설정
node scripts/setup-package-manager.js --global pnpm

# 프로젝트 전용 매니저 설정
node scripts/setup-package-manager.js --project bun

# 사용 가능한 패키지 매니저 목록 확인
node scripts/setup-package-manager.js --list
```

---

## 감지 우선순위

시스템은 다음 순서에 따라 어떤 패키지 매니저를 사용할지 결정합니다:

1. **환경 변수**: `CLAUDE_PACKAGE_MANAGER`
2. **프로젝트 설정**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 필드 정의
4. **락 파일(Lock files)**: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lockb` 존재 여부
5. **전역 설정**: `~/.claude/package-manager.json`
6. **기본값**: 시스템에서 사용 가능한 첫 번째 매니저 (pnpm > bun > yarn > npm 순)

---

## 설정 파일 예시

### 전역 설정 (`~/.claude/package-manager.json`)
```json
{
  "packageManager": "pnpm"
}
```

### 프로젝트 설정 (`.claude/package-manager.json`)
```json
{
  "packageManager": "bun"
}
```

---

## 환경 변수 설정

지정된 패키지 매니저를 강제로 사용하려면 `CLAUDE_PACKAGE_MANAGER` 환경 변수를 설정하십시오:

```powershell
# Windows (PowerShell)
$env:CLAUDE_PACKAGE_MANAGER = "pnpm"

# macOS/Linux (Bash/Zsh)
export CLAUDE_PACKAGE_MANAGER=pnpm
```

**핵심**: Setup-PM 명령어를 통해 프로젝트별로 최적화된 패키지 관리 환경을 구축하고, 다양한 라이브러리 설치 시 발생할 수 있는 호환성 문제를 사전에 방지할 수 있습니다.
