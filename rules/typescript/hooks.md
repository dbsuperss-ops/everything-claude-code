---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# TypeScript/JavaScript 훅 (TypeScript/JavaScript Hooks)

> 이 파일은 [common/hooks.md](../common/hooks.md)을 TypeScript/JavaScript 전용 내용으로 확장합니다.

## PostToolUse 훅

`~/.claude/settings.json`에서 설정하십시오:

- **Prettier**: JS/TS 파일 수정 후 자동 포맷팅
- **TypeScript 체크**: `.ts`/`.tsx` 파일 수정 후 `tsc` 실행
- **console.log 경고**: 수정된 파일에 `console.log`가 있는지 경고

## Stop 훅

- **console.log 검사**: 세션 종료 전 모든 수정된 파일에 `console.log`가 있는지 확인
