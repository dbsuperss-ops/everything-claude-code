---
name: coding-standards
description: TypeScript, JavaScript, React 및 Node.js 개발을 위한 범용 코딩 표준, 최선 관행(Best practices) 및 패턴입니다.
origin: ECC
---

# 코딩 표준 및 최선 관행 (Coding Standards & Best Practices)

모든 프로젝트에 적용 가능한 범용 코딩 표준을 안내합니다.

## 활성화 시점

- 새로운 프로젝트나 모듈을 시작할 때
- 품질 및 유지보수성을 위해 코드를 리뷰할 때
- 컨벤션에 맞춰 기존 코드를 리팩토링할 때
- 명명 규칙(Naming), 포맷팅 또는 구조적 일관성을 강제할 때
- 린팅(Linting), 포맷팅 또는 타입 체크 규칙을 설정할 때
- 새로운 참여자에게 코딩 컨벤션을 안내할 때

## 코드 품질 원칙

### 1. 가독성 우선 (Readability First)
- 코드는 작성되는 시간보다 읽히는 시간이 훨씬 더 많습니다.
- 변수와 함수 이름은 명확하게 지으십시오.
- 주석보다는 스스로를 설명하는 코드(Self-documenting code)를 지향하십시오.
- 일관된 포맷팅을 유지하십시오.

### 2. KISS (Keep It Simple, Stupid)
- 가장 간단하게 작동하는 해결책을 찾으십시오.
- 오버 엔지니어링을 피하십시오.
- 조기 최적화(Premature optimization)를 하지 마십시오.
- 복잡하고 영리한 코드보다 이해하기 쉬운 코드가 더 좋습니다.

### 3. DRY (Don't Repeat Yourself)
- 공통 로직은 함수로 추출하십시오.
- 재사용 가능한 컴포넌트를 만드십시오.
- 유틸리티를 모듈 간에 공유하십시오.
- 복사-붙여넣기 방식의 프로그래밍을 피하십시오.

### 4. YAGNI (You Aren't Gonna Need It)
- 실제로 필요하기 전까지는 기능을 만들지 마십시오.
- 막연한 추측을 바탕으로 하는 범용적인 구현을 피하십시오.
- 꼭 필요한 경우에만 복잡성을 추가하십시오.
- 단순하게 시작하고 필요할 때 리팩토링하십시오.

## TypeScript/JavaScript 표준

### 변수 명명 규칙

```typescript
// ✅ 좋음: 서술적인 이름
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 나쁨: 불분명한 이름
const q = 'election'
const flag = true
const x = 1000
```

### 함수 명명 규칙

```typescript
// ✅ 좋음: 동사-명사 패턴
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 나쁨: 불분명하거나 명사만 사용
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 불변성 패턴 (CRITICAL)

```typescript
// ✅ 항상 스프레드 연산자(Spread operator)를 사용하십시오.
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 절대 직접 수정(Mutate)하지 마십시오.
user.name = 'New Name'  // 나쁨
items.push(newItem)     // 나쁨
```

### 에러 처리

```typescript
// ✅ 좋음: 포괄적인 에러 처리
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}

// ❌ 나쁨: 에러 처리 부재
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### 비동기(Async/Await) 최선 관행

```typescript
// ✅ 좋음: 가능한 경우 병렬로 실행
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 나쁨: 불필요하게 순차적으로 실행
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

## React 최선 관행

### 컴포넌트 구조

```typescript
// ✅ 좋음: 타입이 정의된 함수형 컴포넌트
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}
```

### 상태 관리 (State Management)

```typescript
// ✅ 좋음: 이전 상태를 기반으로 하는 올바른 업데이트
const [count, setCount] = useState(0)

// 함수형 업데이트를 사용하여 이전 상태 보장
setCount(prev => prev + 1)

// ❌ 나쁨: 상태 직접 참조
setCount(count + 1)  // 비동기 시나리오에서 예전 값을 참조할 수 있음
```

### 조건부 렌더링 (Conditional Rendering)

```typescript
// ✅ 좋음: 명확한 조건부 렌더링
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 나쁨: 복잡한 삼항 연산자 남용
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## 파일 구성

### 프로젝트 구조 예시

```
src/
├── app/                    # Next.js 앱 라우터
├── components/            # React 컴포넌트
│   ├── ui/               # 공통 UI 컴포넌트
│   ├── forms/            # 폼 관련 컴포넌트
├── hooks/                # 커스텀 React 훅
├── lib/                  # 유틸리티 및 설정
├── types/                # TypeScript 타입 정의
└── styles/              # 글로벌 스타일
```

### 파일 명명 규칙

```
components/Button.tsx          # 컴포넌트는 PascalCase
hooks/useAuth.ts              # 'use' 접두사와 함께 camelCase
lib/formatDate.ts             # 유틸리티는 camelCase
types/market.types.ts         # .types 접미사와 함께 camelCase
```

## 주석 및 문서화

### 주석 작성 기준

```typescript
// ✅ 좋음: 코드가 '무엇을' 하는지가 아니라 '왜' 그렇게 했는지를 설명
// 장애 상황 시 API 부하를 줄이기 위해 지수 백오프(Exponential backoff)를 사용함
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 대규모 배열의 성능 최적화를 위해 의도적으로 가변(Mutation) 방식 사용함
items.push(newItem)

// ❌ 나쁨: 너무 당연한 내용을 설명
// 카운터를 1 증가시킴
count++
```

## 성능 최선 관행

### 메모이제이션 (Memoization)

```typescript
import { useMemo, useCallback } from 'react'

// ✅ 좋음: 비용이 많이 드는 계산 결과 메모이제이션
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 좋음: 콜백 함수 메모이제이션
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

## 코드 스멜 (Code Smell) 탐지

다음 패턴들을 주의하십시오:

### 1. 너무 긴 함수
- 함수가 50라인을 초과하지 않도록 하십시오.
- 큰 함수는 작은 함수 여러 개로 쪼개십시오.

### 2. 깊은 중첩
- 5단계 이상의 중첩(if, for 등)을 피하십시오.
- Early returns 패턴을 사용하여 코드 깊이를 줄이십시오.

### 3. 매직 넘버 (Magic Numbers)
- 의미를 알 수 없는 숫자를 직접 사용하지 마십시오.
- 대신 명명된 상수(Named constants)를 사용하십시오.

**기억하십시오**: 코드 품질은 타협의 대상이 아닙니다. 깨끗하고 유지보수 가능한 코드는 빠른 개발과 자신감 있는 리팩토링을 가능하게 합니다.
    
