---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# TypeScript/JavaScript 코딩 스타일 (TypeScript/JavaScript Coding Style)

> 이 파일은 [common/coding-style.md](../common/coding-style.md)을 TypeScript/JavaScript 전용 내용으로 확장합니다.

## 불변성 (Immutability)

불변 업데이트를 위해 스프레드(Spread) 연산자를 사용하십시오:

```typescript
// 나쁨: 수정(Mutation)
function updateUser(user, name) {
  user.name = name  // 직접 수정!
  return user
}

// 좋음: 불변성(Immutability)
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## 에러 처리

try-catch와 함께 async/await를 사용하십시오:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('작업 실패:', error)
  throw new Error('사용자 친화적인 상세 메시지')
}
```

## 입력값 검증

스키마 기반 검증을 위해 Zod를 사용하십시오:

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## Console.log

- 프로덕션 코드에 `console.log` 문을 남기지 마십시오.
- 대신 적절한 로깅 라이브러리를 사용하십시오.
- 자동 감지를 위해 훅(Hooks) 설정을 참조하십시오.
