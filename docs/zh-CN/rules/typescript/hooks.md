---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---

# TypeScript/JavaScript 후크 (Hooks)

> 이 문서는 [common/hooks.md](../common/hooks.md)의 내용을 바탕으로 TypeScript/JavaScript에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에 다음 항목을 구성하십시오:

* **Prettier**: JS/TS 파일 편집 후 자동으로 포매팅을 수행합니다.
* **TypeScript 검사**: `.ts`/`.tsx` 파일 편집 후 `tsc`를 실행하여 타입을 점검합니다.
* **console.log 경고**: 수정된 파일 내에 `console.log`가 포함되어 있으면 경고를 표시합니다.

## Stop 후크

* **console.log 감사(Audit)**: 세션 종료 전, 모든 수정된 파일 에 `console.log`가 남아있는지 최종 확인합니다.
