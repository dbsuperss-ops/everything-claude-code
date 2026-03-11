---
description: "공통 규칙을 확장하는 TypeScript 코딩 스타일"
globs: ["**/*.ts", "**/*.tsx", "**/*.js", "**/*.jsx"]
alwaysApply: false
---
# TypeScript/JavaScript 코딩 스타일 (Coding Style)

> 이 문서는 공통 코딩 스타일 규칙을 기반으로 TypeScript/JavaScript에 특화된 내용을 확장합니다.

## 불변성 (Immutability)

불변 상태 업데이트를 위해 전개 연산자(Spread operator)를 사용하십시오:

```typescript
// 잘못된 예: 상태 변경(Mutation)
function updateUser(user, name) {
  user.name = name  // 직접 수정 금지!
  return user
}

// 올바른 예: 불변성 유지
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## 에러 처리

`try-catch`와 함께 `async/await`를 사용하십시오:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('작업 실패:', error)
  throw new Error('사용자 친화적인 상세 오류 메시지')
}
```

## 입력값 검증

스키마 기반 검증을 위해 **Zod**를 사용하십시오:

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## Console.log 금지

- 프로덕션 코드에서는 `console.log` 사용이 허용되지 않습니다.
- 대신 적절한 로깅 라이브러리를 사용하십시오.
- 자동 감지를 위해 후크(Hooks) 설정 규칙을 확인하십시오.
