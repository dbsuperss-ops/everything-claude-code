---
name: coding-standards
description: TypeScript, JavaScript, React, Node.js 개발에 적용되는 공통 코딩 표준, 베스트 프랙티스 및 패턴입니다.
origin: ECC
---

# 코딩 표준 및 베스트 프랙티스

모든 프로젝트에 적용되는 범용적인 코딩 표준입니다.

## 적용 시점

* 새로운 프로젝트나 모듈을 시작할 때
* 코드 품질 및 유지보수성을 검토할 때
* 기존 코드를 규약에 맞춰 리팩토링할 때
* 명명 규칙, 포맷 또는 구조의 일관성을 강제해야 할 때
* 린팅(Linting), 포매팅, 타입 체크 규칙을 설정할 때
* 새로운 기여자에게 코딩 규칙을 안내할 때

## 코드 품질 원칙

### 1. 가독성 우선
* 코드는 작성되는 횟수보다 읽히는 횟수가 훨씬 많습니다.
* 변수와 함수 이름은 명확하게 지으십시오.
* 주석보다는 코드 자체로 의도가 드러나는 '자기 문서화(Self-documenting)' 코드를 지향하십시오.
* 일관된 포매팅을 유지하십시오.

### 2. KISS (Keep It Simple, Stupid)
* 작동하는 가장 간단한 해결책을 선택하십시오.
* 오버엔지니어링(Over-engineering)을 피하십시오.
* 조기 최적화(Premature optimization)를 하지 마십시오.
* 화려하고 복잡한 코드보다 이해하기 쉬운 코드가 더 좋습니다.

### 3. DRY (Don't Repeat Yourself)
* 공통 로직은 함수로 추출하십시오.
* 재사용 가능한 컴포넌트를 만드십시오.
* 유틸리티 함수는 모듈 간에 공유하십시오.
* '복사-붙여넣기' 식의 프로그래밍을 피하십시오.

### 4. YAGNI (You Ain't Gonna Need It)
* 당장 필요하지 않은 기능을 미리 구현하지 마십시오.
* 추측에 근거한 범용화(Speculative generalization)를 피하십시오.
* 실제로 필요한 시점에만 복잡성을 추가하십시오.
* 간단하게 시작하고, 필요할 때 리팩토링하십시오.

## TypeScript/JavaScript 표준

### 변수 명명
```typescript
// ✅ 좋은 예: 서술적인 이름
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 나쁜 예: 불분명한 이름
const q = 'election'
const flag = true
const x = 1000
```

### 함수 명명
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

### 불변성(Immutability) 패턴 (중요)
```typescript
// ✅ 항상 전개 연산자(Spread operator)를 사용하십시오
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 절대 데이터를 직접 수정(Mutate)하지 마십시오
user.name = 'New Name'  // 금지
items.push(newItem)     // 금지
```

### 에러 처리
```typescript
// ✅ 좋은 예: 종합적인 에러 처리
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('Fetch 실패:', error)
    throw new Error('데이터를 가져오는데 실패했습니다')
  }
}

// ❌ 나쁜 예: 에러 처리 없음
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Await 베스트 프랙티스
```typescript
// ✅ 좋은 예: 가능한 경우 병렬 실행
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

### 타입 안정성
```typescript
// ✅ 좋은 예: 적절한 타입 지정
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

## React 베스트 프랙티스

### 컴포넌트 구조
```typescript
// ✅ 좋은 예: 타입이 지정된 함수형 컴포넌트
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

// ❌ 나쁜 예: 타입 부재, 불명확한 구조
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### 커스텀 훅 (Custom Hooks)
```typescript
// ✅ 좋은 예: 재사용 가능한 커스텀 훅
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
setCount(count + 1)  // 비동기 상황에서 오래된(Stale) 값을 참조할 수 있음
```

### 조건부 렌더링
```typescript
// ✅ 좋은 예: 명확한 조건부 렌더링
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 나쁜 예: 복잡한 삼항 연산자 중첩
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API 설계 표준

### REST API 규칙
```
GET    /api/markets              # 모든 마켓 목록 조회
GET    /api/markets/:id          # 특정 마켓 조회
POST   /api/markets              # 새로운 마켓 생성
PUT    /api/markets/:id          # 마켓 수정 (전체 교체)
PATCH  /api/markets/:id          # 마켓 수정 (일부 수정)
DELETE /api/markets/:id          # 마켓 삭제

# 필터링을 위한 쿼리 파라미터 활용
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

// 성공 응답 예시
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// 에러 응답 예시
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
    // 검증된 데이터를 사용하여 로직 진행
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: '검증 실패',
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
│   ├── markets/           # 마켓 관련 페이지
│   └── (auth)/           # 인증 관련 페이지 (라우트 그룹)
├── components/            # React 컴포넌트
│   ├── ui/               # 공통 UI 컴포넌트
│   ├── forms/            # 폼 관련 컴포넌트
│   └── layouts/          # 레이아웃 컴포넌트
├── hooks/                # 커스텀 React 훅
├── lib/                  # 유틸리티 및 설정
│   ├── api/             # API 클라이언트
│   ├── utils/           # 헬퍼 함수
│   └── constants/       # 상수
├── types/                # TypeScript 타입 정의
└── styles/              # 글로벌 스타일
```

### 파일 명명 규칙
```
components/Button.tsx          # 컴포넌트는 PascalCase
hooks/useAuth.ts              # 훅은 camelCase이며 'use' 접두사 사용
lib/formatDate.ts             # 유틸리티는 camelCase
types/market.types.ts         # 타입은 camelCase이며 .types 접미사 사용
```

## 주석 및 문서화

### 주석 작성 기준
```typescript
// ✅ 좋은 예: '무엇'이 아닌 '왜'를 설명하십시오
// 장애 상황 발생 시 API 과부하를 방지하기 위해 지수 백오프(Exponential backoff)를 사용합니다.
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 대규모 배열 처리를 위해 성능상 의도적으로 데이터를 직접 수정(Mutation)합니다.
items.push(newItem)

// ❌ 나쁜 예: 뻔한 내용 나열
// 카운터를 1 증가시킵니다
count++

// 이름을 사용자의 이름으로 설정합니다
name = user.name
```

### 공개 API를 위한 JSDoc
```typescript
/**
 * 의미론적 유사성을 사용하여 마켓을 검색합니다.
 *
 * @param query - 자연어 검색 쿼리
 * @param limit - 최대 결과 수 (기본값: 10)
 * @returns 유사성 점수 순으로 정렬된 마켓 배열
 * @throws {Error} OpenAI API 실패 또는 Redis 사용 불가 시 에러 발생
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

## 성능 베스트 프랙티스

### 메모이제이션 (Memoization)
```typescript
import { useMemo, useCallback } from 'react'

// ✅ 좋은 예: 비용이 많이 드는 연산 결과 캐싱
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 좋은 예: 콜백 함수 캐싱
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

// ❌ 나쁜 예: 모든 컬럼 선택 (*)
const { data } = await supabase
  .from('markets')
  .select('*')
```

## 테스트 표준

### 테스트 구조 (AAA 패턴)
```typescript
test('유사도가 올바르게 계산되어야 함', () => {
  // 1. Arrange (준비)
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // 2. Act (실행)
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // 3. Assert (검증)
  expect(similarity).toBe(0)
})
```

### 테스트 명명 규칙
```typescript
// ✅ 좋은 예: 서술적인 테스트 이름
test('검색 결과가 없을 때 빈 배열을 반환해야 함', () => { })
test('OpenAI API 키가 없을 때 에러를 발생시켜야 함', () => { })
test('Redis를 사용할 수 없을 때 부분 문자열 검색으로 대체되어야 함', () => { })

// ❌ 나쁜 예: 모호한 테스트 이름
test('작동함', () => { })
test('검색 테스트', () => { })
```

## 코드 품질 저하 신호 (Code Smell)

다음과 같은 안티 패턴을 경계하십시오:

### 1. 너무 긴 함수
```typescript
// ❌ 나쁜 예: 50줄이 넘는 함수
function processMarketData() {
  // 100줄의 코드...
}

// ✅ 좋은 예: 작은 함수들로 분리
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 깊은 중첩 (Nested Conditions)
```typescript
// ❌ 나쁜 예: 5단계 이상의 중첩
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 로직 실행
        }
      }
    }
  }
}

// ✅ 좋은 예: Early Returns (빠른 반환) 활용
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 로직 실행
```

### 3. 매직 넘버 (Magic Numbers)
```typescript
// ❌ 나쁜 예: 설명 없는 숫자 사용
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ 좋은 예: 이름이 있는 상수 활용
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**핵심**: 코드 품질은 타협의 대상이 아닙니다. 명확하고 유지보수 가능한 코드는 빠른 개발과 자신감 있는 리팩토링을 가능하게 합니다.
