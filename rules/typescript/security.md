---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# TypeScript/JavaScript 보안 (TypeScript/JavaScript Security)

> 이 파일은 [common/security.md](../common/security.md)을 TypeScript/JavaScript 전용 내용으로 확장합니다.

## 비밀 정보 관리

```typescript
// 나쁨: 하드코딩된 비밀 정보
const apiKey = "sk-proj-xxxxx"

// 좋음: 환경 변수 사용
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY가 설정되지 않았습니다.')
}
```

## 에이전트 지원

- 포괄적인 보안 감사를 위해 `security-reviewer` 스킬을 활용하십시오.
