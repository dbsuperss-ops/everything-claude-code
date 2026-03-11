---
description: "공통 규칙을 확장하는 TypeScript 후크(Hooks)"
globs: ["**/*.ts", "**/*.tsx", "**/*.js", "**/*.jsx"]
alwaysApply: false
---
# TypeScript/JavaScript 후크 (Hooks)

> 이 문서는 공통 후크 규칙을 기반으로 TypeScript/JavaScript에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에서 다음 항목을 구성하십시오:

- **Prettier**: JS/TS 파일 편집 후 자동 포매팅 수행
- **TypeScript check**: `.ts`/`.tsx` 파일 편집 후 `tsc` 실행
- **console.log warning**: 편집된 파일에 `console.log`가 포함된 경우 경고

## Stop 후크

- **console.log audit**: 세션 종료 전 모든 수정된 파일에서 `console.log` 존재 여부를 확인합니다.
