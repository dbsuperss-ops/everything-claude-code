---
description: "공통 규칙을 확장하는 TypeScript 보안"
globs: ["**/*.ts", "**/*.tsx", "**/*.js", "**/*.jsx"]
alwaysApply: false
---
# TypeScript/JavaScript 보안 (Security)

> 이 문서는 공통 보안 규칙을 기반으로 TypeScript/JavaScript에 특화된 내용을 확장합니다.

## 비밀 정보(Secret) 관리

```typescript
// 절대 금지: 하드코딩된 비밀 정보
const apiKey = "sk-proj-xxxxx"

// 권장 사항: 환경 변수 사용
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY가 구성되지 않았습니다')
}
```

## 에이전트 지원

- 포괄적인 보안 감사를 위해 **security-reviewer** 스킬을 활용하십시오.
