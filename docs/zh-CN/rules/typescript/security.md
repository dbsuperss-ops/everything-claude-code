---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---

# TypeScript/JavaScript 보안 (Security)

> 이 문서는 [common/security.md](../common/security.md)의 내용을 바탕으로 TypeScript/JavaScript에 특화된 내용을 확장합니다.

## 비밀 정보(Secrets) 관리

```typescript
// 절대 금지: 비밀 정보 하드코딩
const apiKey = "sk-proj-xxxxx"

// 항상 준수: 환경 변수 사용
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY가 설정되지 않았습니다')
}
```

## 에이전트 지원

* 종합적인 보안 감사를 위해 **security-reviewer** 스킬을 활용하십시오.
