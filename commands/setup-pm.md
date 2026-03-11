---
이름: setup-pm
설명: 선호하는 패키지 매니저 (npm/pnpm/yarn/bun)를 설정합니다.
에이전트 호출 비활성화: true
---

# 패키지 매니저 설정 (Package Manager Setup)

프로젝트 또는 전역적으로 선호하는 패키지 매니저를 설정합니다.

## 사용법

```bash
# 현재 패키지 매니저 감지
node scripts/setup-package-manager.js --detect

# 전역 설정값 설정
node scripts/setup-package-manager.js --global pnpm

# 프로젝트별 설정값 설정
node scripts/setup-package-manager.js --project bun

# 사용 가능한 패키지 매니저 목록 확인
node scripts/setup-package-manager.js --list
```

## 감지 우선순위 (Detection Priority)

사용할 패키지 매니저를 결정할 때 다음 순서대로 확인합니다:

1. **환경 변수**: `CLAUDE_PACKAGE_MANAGER`
2. **프로젝트 설정**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 필드
4. **락 파일 (Lock file)**: package-lock.json, yarn.lock, pnpm-lock.yaml, 또는 bun.lockb의 존재 여부
5. **전역 설정**: `~/.claude/package-manager.json`
6. **폴백 (Fallback)**: 사용 가능한 첫 번째 패키지 매니저 (pnpm > bun > yarn > npm)

## 설정 파일

### 전역 설정 (Global Configuration)
```json
// ~/.claude/package-manager.json
{
  "packageManager": "pnpm"
}
```

### 프로젝트 설정 (Project Configuration)
```json
// .claude/package-manager.json
{
  "packageManager": "bun"
}
```

### package.json
```json
{
  "packageManager": "pnpm@8.6.0"
}
```

## 환경 변수

다른 모든 감지 방법을 무시하도록 `CLAUDE_PACKAGE_MANAGER`를 설정할 수 있습니다:

```bash
# Windows (PowerShell)
$env:CLAUDE_PACKAGE_MANAGER = "pnpm"

# macOS/Linux
export CLAUDE_PACKAGE_MANAGER=pnpm
```

## 감지 실행

현재 패키지 매니저 감지 결과를 확인하려면 다음을 실행하십시오:

```bash
node scripts/setup-package-manager.js --detect
```
