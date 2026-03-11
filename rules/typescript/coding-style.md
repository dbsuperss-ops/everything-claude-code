---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
---
# TypeScript/JavaScript 코딩 스타일 (Coding Style)

> 이 파일은 [common/coding-style.md](../common/coding-style.md)의 내용을 TypeScript/JavaScript 전용 콘텐츠로 확장합니다.

## 타입 및 인터페이스 (Types and Interfaces)

공개 API, 공유 모델 및 컴포넌트 Props를 명시적이고 읽기 쉬우며 재사용 가능하게 만들기 위해 타입을 사용하십시오.

### 공개 API (Public APIs)

- 내보낸(export) 함수, 공유 유틸리티 및 공개 클래스 메서드에 매개변수 및 반환 타입을 추가하십시오.
- 명확한 지역 변수 타입은 TypeScript가 추론하도록 하십시오.
- 반복되는 인라인 객체 구조는 명명된 타입(Type)이나 인터페이스(Interface)로 추출하십시오.

```typescript
// ❌ 잘못된 예: 명시적 타입이 없는 내보낸 함수
export function formatUser(user) {
  return `${user.firstName} ${user.lastName}`
}

// ✅ 올바른 예: 공개 API에 명시적 타입 적용
interface User {
  firstName: string
  lastName: string
}

export function formatUser(user: User): string {
  return `${user.firstName} ${user.lastName}`
}
```

### 인터페이스 vs. 타입 별칭 (Interfaces vs. Type Aliases)

- 확장하거나 구현(Implement)할 수 있는 객체 구조에는 `interface`를 사용하십시오.
- 유니온(Union), 인터섹션(Intersection), 튜플, 매핑된 타입 및 유틸리티 타입에는 `type`을 사용하십시오.
- 상호 운용성을 위해 `enum`이 반드시 필요한 경우가 아니라면, `enum` 대신 문자열 리터럴 유니온을 선호하십시오.

```typescript
interface User {
  id: string
  email: string
}

type UserRole = 'admin' | 'member'
type UserWithRole = User & {
  role: UserRole
}
```

### `any` 금지

- 애플리케이션 코드에서 `any` 사용을 피하십시오.
- 외부 또는 신뢰할 수 없는 입력에는 `unknown`을 사용하고, 안전하게 타입을 좁혀서(Narrowing) 사용하십시오.
- 값의 타입이 호출자에 따라 달라지는 경우 제네릭(Generics)을 사용하십시오.

```typescript
// ❌ 잘못된 예: any는 타입 안전성을 제거함
function getErrorMessage(error: any) {
  return error.message
}

// ✅ 올바른 예: unknown은 안전한 타입 좁히기를 강제함
function getErrorMessage(error: unknown): string {
  if (error instanceof Error) {
    return error.message
  }

  return 'Unexpected error'
}
```

### React Props

- 컴포넌트 Props는 명명된 `interface` 또는 `type`으로 정의하십시오.
- 콜백(Callback) Props의 타입을 명시적으로 지정하십시오.
- 특별한 이유가 없는 한 `React.FC`를 사용하지 마십시오.

```typescript
interface User {
  id: string
  email: string
}

interface UserCardProps {
  user: User
  onSelect: (id: string) => void
}

function UserCard({ user, onSelect }: UserCardProps) {
  return <button onClick={() => onSelect(user.id)}>{user.email}</button>
}
```

### JavaScript 파일

- `.js` 및 `.jsx` 파일에서, 타입 표시가 명확성을 높이지만 TypeScript 마이그레이션이 실질적으로 어려운 경우 JSDoc을 사용하십시오.
- JSDoc을 런타임 동작과 일치시키십시오.

```javascript
/**
 * @param {{ firstName: string, lastName: string }} user
 * @returns {string}
 */
export function formatUser(user) {
  return `${user.firstName} ${user.lastName}`
}
```

## 불변성 (Immutability)

불변 업데이트를 위해 스프레드(Spread) 연산자를 사용하십시오:

```typescript
interface User {
  id: string
  name: string
}

// ❌ 잘못된 예: 직접 수정 (Mutation)
function updateUser(user: User, name: string): User {
  user.name = name // 원본 변경!
  return user
}

// ✅ 올바른 예: 불변성 유지
function updateUser(user: Readonly<User>, name: string): User {
  return {
    ...user,
    name
  }
}
```

## 에러 처리 (Error Handling)

async/await와 try-catch를 사용하고, `unknown` 에러의 타입을 안전하게 좁히십시오:

```typescript
interface User {
  id: string
  email: string
}

declare function riskyOperation(userId: string): Promise<User>

function getErrorMessage(error: unknown): string {
  if (error instanceof Error) {
    return error.message
  }

  return 'Unexpected error'
}

const logger = {
  error: (message: string, error: unknown) => {
    // 실제 운영 환경의 로거(pino, winston 등)로 대체하십시오.
  }
}

async function loadUser(userId: string): Promise<User> {
  try {
    const result = await riskyOperation(userId)
    return result
  } catch (error: unknown) {
    logger.error('Operation failed', error)
    throw new Error(getErrorMessage(error))
  }
}
```

## 입력값 검증 (Input Validation)

스키마 기반 검증을 위해 Zod를 사용하고 스키마로부터 타입을 추론하십시오:

```typescript
import { z } from 'zod'

const userSchema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

type UserInput = z.infer<typeof userSchema>

const validated: UserInput = userSchema.parse(input)
```

## Console.log

- 프로덕션 코드에 `console.log` 문을 남기지 마십시오.
- 대신 적절한 로깅 라이브러리를 사용하십시오.
- 자동 감지를 위해 후크(Hooks)를 확인하십시오.
