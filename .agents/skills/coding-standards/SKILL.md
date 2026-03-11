---
name: coding-standards
description: TypeScript, JavaScript, React 및 Node.js 개발을 위한 범용 코딩 표준, 모범 사례 및 패턴입니다.
origin: ECC
---

# 코딩 표준 및 모범 사례 (Coding Standards & Best Practices)

모든 프로젝트에 적용 가능한 범용적인 코딩 표준입니다.

## 활성화 시기

- 새로운 프로젝트나 모듈을 시작할 때
- 품질 및 유지보수성을 위해 코드를 검토할 때
- 관례를 따르도록 기존 코드를 리팩토링할 때
- 명명, 포매팅 또는 구조적 일관성을 강제할 때
- 린팅(Linting), 포매팅 또는 타입 체크 규칙을 설정할 때
- 새로운 기여자에게 코딩 관례를 안내(Onboarding)할 때

## 코드 품질 원칙

### 1. 가독성 우선 (Readability First)
- 코드는 작성되는 시간보다 읽히는 시간이 더 많습니다.
- 변수와 함수 이름은 명확하게 지으십시오.
- 주석보다는 그 자체로 설명이 되는 코드(Self-documenting code)를 선호하십시오.
- 일관된 포매팅을 유지하십시오.

### 2. KISS (Keep It Simple, Stupid)
- 작동하는 가장 간단한 해결책을 찾으십시오.
- 과도한 엔지니어링(Over-engineering)을 피하십시오.
- 조기 최적화(Premature optimization)를 하지 마십시오.
- 똑똑해 보이는 코드보다는 이해하기 쉬운 코드가 더 좋습니다.

### 3. DRY (Don't Repeat Yourself)
- 공통 로직은 함수로 추출하십시오.
- 재사용 가능한 컴포넌트를 만드십시오.
- 모듈 간에 유틸리티를 공유하십시오.
- 복사-붙여넣기 방식의 프로그래밍을 피하십시오.

### 4. YAGNI (You Aren't Gonna Need It)
- 실제로 필요하기 전까지는 기능을 만들지 마십시오.
- 추측에 근거한 범용성(Speculative generality)을 피하십시오.
- 꼭 필요한 경우에만 복잡성을 추가하십시오.
- 단순하게 시작하고, 필요할 때 리팩토링하십시오.

## TypeScript/JavaScript 표준

### 변수 명명 (Variable Naming)

```typescript
// ✅ 좋은 예: 설명적인 이름
const marketSearchQuery = '선거'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 나쁜 예: 불분명한 이름
const q = '선거'
const flag = true
const x = 1000
```

### 함수 명명 (Function Naming)

```typescript
// ✅ 좋은 예: 동사-명사 패턴
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 나쁜 예: 불분명하거나 명사만 사용
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 불변성 패턴 (CRITICAL)

```typescript
// ✅ 항상 전개 연산자(Spread operator)를 사용하십시오
const updatedUser = {
  ...user,
  name: '새 이름'
}

const updatedArray = [...items, newItem]

// ❌ 절대 직접 수정(Mutate)하지 마십시오
user.name = '새 이름'  // 나쁨
items.push(newItem)     // 나쁨
```

### 에러 처리

```typescript
// ✅ 좋은 예: 포괄적인 에러 처리
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('데이터 가져오기 실패:', error)
    throw new Error('데이터를 가져오지 못했습니다')
  }
}

// ❌ 나쁜 예: 에러 처리 없음
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Await 모범 사례

```typescript
// ✅ 좋은 예: 가능하면 병렬로 실행
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 나쁜 예: 불필요하게 순차적으로 실행
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 타입 안전성 (Type Safety)

```typescript
// ✅ 좋은 예: 적절한 타입 정의
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // 구현부
}

// ❌ 나쁜 예: 'any' 사용
function getMarket(id: any): Promise<any> {
  // 구현부
}
```

## React 모범 사례

### 컴포넌트 구조

```typescript
// ✅ 좋은 예: 타입이 포함된 함수형 컴포넌트
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

// ❌ 나쁜 예: 타입 부재, 불분명한 구조
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### 커스텀 후크 (Custom Hooks)

```typescript
// ✅ 좋은 예: 재사용 가능한 커스텀 후크
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// 사용 예시
const debouncedQuery = useDebounce(searchQuery, 500)
```

### 상태 관리

```typescript
// ✅ 좋은 예: 올바른 상태 업데이트
const [count, setCount] = useState(0)

// 이전 상태를 기반으로 업데이트할 때는 함수형 업데이트 사용
setCount(prev => prev + 1)

// ❌ 나쁜 예: 상태 직접 참조
setCount(count + 1)  // 비동기 상황에서 값이 부정확할 수 있음
```

### 조건부 렌더링

```typescript
// ✅ 좋은 예: 명확한 조건부 렌더링
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 나쁜 예: 복잡한 삼항 연산자
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API 설계 표준

### REST API 관례

```
GET    /api/markets              # 모든 마켓 목록 조회
GET    /api/markets/:id          # 특정 마켓 정보 조회
POST   /api/markets              # 새로운 마켓 생성
PUT    /api/markets/:id          # 마켓 수정 (전체 교체)
PATCH  /api/markets/:id          # 마켓 수정 (일부 수정)
DELETE /api/markets/:id          # 마켓 삭제

# 필터링을 위한 쿼리 파라미터
GET /api/markets?status=active&limit=10&offset=0
```

### 응답 형식

```typescript
// ✅ 좋은 예: 일관된 응답 구조
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}

// 성공 응답
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// 에러 응답
return NextResponse.json({
  success: false,
  error: '잘못된 요청입니다'
}, { status: 400 })
```

### 입력값 검증

```typescript
import { z } from 'zod'

// ✅ 좋은 예: 스키마 검증
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1)
})

export async function POST(request: Request) {
  const body = await request.json()

  try {
    const validated = CreateMarketSchema.parse(body)
    // 검증된 데이터로 작업 진행
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: '유효성 검사 실패',
        details: error.errors
      }, { status: 400 })
    }
  }
}
```

## 파일 구성

### 프로젝트 구조

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API 라우트
│   ├── markets/           # 마켓 페이지들
│   └── (auth)/           # 인증 페이지들 (라우트 그룹)
├── components/            # React 컴포넌트
│   ├── ui/               # 범용 UI 컴포넌트
│   ├── forms/            # 폼 관련 컴포넌트
│   └── layouts/          # 레이아웃 관련 컴포넌트
├── hooks/                # 커스텀 React 후크
├── lib/                  # 유틸리티 및 설정 파일
│   ├── api/             # API 클라이언트
│   ├── utils/           # 헬퍼 함수
│   └── constants/       # 상수
├── types/                # TypeScript 타입 정의
└── styles/              # 글로벌 스타일
```

### 파일 명명 규칙

```
components/Button.tsx          # 컴포넌트에는 PascalCase 사용
hooks/useAuth.ts              # 'use' 접두사와 함께 camelCase 사용
lib/formatDate.ts             # 유틸리티에는 camelCase 사용
types/market.types.ts         # .types 접미사와 함께 camelCase 사용
```

## 주석 및 문서화

### 주석 작성 시점

```typescript
// ✅ 좋은 예: ‘무엇을’이 아니라 ‘왜’ 하는지 설명하십시오
// 장애 상황에서 API 부하를 방지하기 위해 지수 백오프를 사용합니다
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 대규모 배열 작업 시 성능 향상을 위해 여기서는 의도적으로 직접 수정을 사용합니다
items.push(newItem)

// ❌ 나쁜 예: 뻔한 내용 설명
// 카운터를 1 증가시킵니다
count++

// 이름을 사용자의 이름으로 설정합니다
name = user.name
```

### 공용 API를 위한 JSDoc

```typescript
/**
 * 의미적 유사성을 사용하여 마켓을 검색합니다.
 *
 * @param query - 자연어 검색 쿼리
 * @param limit - 최대 결과 수 (기본값: 10)
 * @returns 유사도 점수에 따라 정렬된 마켓 배열
 * @throws {Error} OpenAI API 장애 또는 Redis 사용 불가 시 에러 발생
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('선거', 5)
 * console.log(results[0].name) // "트럼프 vs 바이든"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10
): Promise<Market[]> {
  // 구현부
}
```

## 성능 모범 사례

### 메모이제이션 (Memoization)

```typescript
import { useMemo, useCallback } from 'react'

// ✅ 좋은 예: 비용이 큰 계산 결과 메모이징
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 좋은 예: 콜백 함수 메모이징
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 지연 로딩 (Lazy Loading)

```typescript
import { lazy, Suspense } from 'react'

// ✅ 좋은 예: 무거운 컴포넌트 지연 로딩
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### 데이터베이스 쿼리

```typescript
// ✅ 좋은 예: 필요한 컬럼만 선택
const { data } = await supabase
  .from('markets')
  .select('id, name, status')
  .limit(10)

// ❌ 나쁜 예: 모든 컬럼(*) 선택
const { data } = await supabase
  .from('markets')
  .select('*')
```

## 테스트 표준

### 테스트 구조 (AAA 패턴)

```typescript
test('유사도를 정확하게 계산함', () => {
  // Arrange (준비)
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act (실행)
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert (검증)
  expect(similarity).toBe(0)
})
```

### 테스트 명명 규칙

```typescript
// ✅ 좋은 예: 설명적인 테스트 이름
test('쿼리와 일치하는 마켓이 없을 때 빈 배열을 반환함', () => { })
test('OpenAI API 키가 없을 때 에러를 발생시킴', () => { })
test('Redis를 사용할 수 없을 때 부분 문자열 검색으로 대체함', () => { })

// ❌ 나쁜 예: 모호한 테스트 이름
test('작동함', () => { })
test('검색 테스트', () => { })
```

## 코드 스멜(Code Smell) 감지

다음과 같은 안티 패턴을 주의하십시오:

### 1. 긴 함수
```typescript
// ❌ 나쁜 예: 50줄이 넘어가는 함수
function processMarketData() {
  // 100줄의 코드
}

// ✅ 좋은 예: 작은 함수들로 분리
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 깊은 중첩
```typescript
// ❌ 나쁜 예: 5단계 이상의 중첩
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 작업 수행
        }
      }
    }
  }
}

// ✅ 좋은 예: 이른 반환 (Early returns)
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 작업 수행
```

### 3. 매직 넘버 (Magic Numbers)
```typescript
// ❌ 나쁜 예: 설명 없는 숫자 사용
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ 좋은 예: 명명된 상수 사용
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**기억해 두세요**: 코드 품질은 타협할 대상이 아닙니다. 명확하고 유지보수가 쉬운 코드는 빠른 개발과 자신감 있는 리팩토링을 가능하게 합니다.
