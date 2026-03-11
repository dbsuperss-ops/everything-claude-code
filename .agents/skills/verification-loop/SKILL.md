---
name: verification-loop
description: "Claude Code 세션을 위한 포괄적인 검증 시스템입니다."
origin: ECC
---

# 검증 루프 스킬 (Verification Loop Skill)

Claude Code 세션을 위한 포괄적인 검증 시스템입니다.

## 사용 시기

다음 상황에서 이 스킬을 실행하십시오:
- 기능 구현 또는 상당한 코드 변경을 완료한 후
- PR(Pull Request)을 생성하기 전
- 품질 게이트(Quality gates) 통과를 보장하고 싶을 때
- 리팩토링을 마친 후

## 검증 단계

### 1단계: 빌드 검증
```bash
# 프로젝트 빌드 확인
npm run build 2>&1 | tail -20
# 또는
pnpm build 2>&1 | tail -20
```

빌드가 실패할 경우, 계속 진행하기 전에 작업을 중단(STOP)하고 수정하십시오.

### 2단계: 타입 체크 (Type Check)
```bash
# TypeScript 프로젝트
npx tsc --noEmit 2>&1 | head -30

# Python 프로젝트
pyright . 2>&1 | head -30
```

모든 타입 에러를 보고하십시오. 치명적인 에러는 계속 진행하기 전에 수정해야 합니다.

### 3단계: 린트 체크 (Lint Check)
```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### 4단계: 테스트 슈트 실행
```bash
# 커버리지와 함께 테스트 실행
npm run test -- --coverage 2>&1 | tail -50

# 커버리지 임계값 확인
# 목표: 최소 80%
```

다음 항목을 보고하십시오:
- 총 테스트 수: X
- 통과: X
- 실패: X
- 커버리지: X%

### 5단계: 보안 스캔
```bash
# 시크릿(Secrets) 노출 확인
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# console.log 확인
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### 6단계: 변경 사항(Diff) 검토
```bash
# 변경된 내용 확인
git diff --stat
git diff HEAD~1 --name-only
```

각 변경된 파일에서 다음을 검토하십시오:
- 의도치 않은 변경 사항
- 누락된 에러 처리 로직
- 잠재적인 에제 케이스(Edge cases)

## 출력 형식

모든 단계를 실행한 후 검증 보고서를 작성하십시오:

```
검증 보고서 (VERIFICATION REPORT)
=================================

빌드:     [통과/실패]
타입:     [통과/실패] (X개의 에러)
린트:     [통과/실패] (X개의 경고)
테스트:   [통과/실패] (X/Y 통과, Z% 커버리지)
보안:     [통과/실패] (X개의 이슈)
변경사항: [X개의 파일 변경됨]

전체 상태: PR 생성 [준비됨/준비되지 않음]

수정할 이슈:
1. ...
2. ...
```

## 지속 모드

긴 세션의 경우, 15분마다 또는 주요 변경 사항이 있을 때마다 검증을 수행하십시오:

```markdown
정신적인 체크포인트를 설정하십시오:
- 각 함수 작성을 마칠 때마다
- 컴포넌트 하나를 완성할 때마다
- 다음 작업으로 넘어가기 전

실행: /verify
```

## 후크(Hooks)와의 통합

이 스킬은 PostToolUse 후크를 보완하며 더 깊이 있는 검증을 제공합니다.
후크는 이슈를 즉각적으로 포착하며, 이 스킬은 포괄적인 검토를 제공합니다.
