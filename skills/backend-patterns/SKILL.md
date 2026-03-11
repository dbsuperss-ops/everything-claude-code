---
name: backend-patterns
description: Node.js, Express, Next.js API 라우트를 위한 백엔드 아키텍처 패턴, API 설계, 데이터베이스 최적화 및 서버측 최선 관행(Best practices)입니다.
origin: ECC
---

# 백엔드 개발 패턴 (Backend Development Patterns)

확장 가능한 서버측 애플리케이션을 위한 백엔드 아키텍처 패턴과 최선 관행을 안내합니다.

## 활성화 시점

- REST 또는 GraphQL API 엔드포인트 설계 시
- 리포지토리(Repository), 서비스(Service) 또는 컨트롤러(Controller) 레이어 구현 시
- 데이터베이스 쿼리 최적화 시 (N+1 문제, 인덱싱, 커넥션 풀링 등)
- 캐싱 추가 시 (Redis, 인메모리, HTTP 캐시 헤더 등)
- 백그라운드 작업 또는 비동기 처리 설정 시
- API를 위한 에러 처리 및 유효성 검사 구조 설계 시
- 미들웨어 구축 시 (인증, 로깅, 속도 제한 등)

## API 설계 패턴

### RESTful API 구조

```typescript
// ✅ 리소스 기반 URL
GET    /api/markets                 # 리소스 목록 조회
GET    /api/markets/:id             # 단일 리소스 조회
POST   /api/markets                 # 리소스 생성
PUT    /api/markets/:id             # 리소스 전체 교체
PATCH  /api/markets/:id             # 리소스 일부 수정
DELETE /api/markets/:id             # 리소스 삭제

// ✅ 필터링, 정렬, 페이지네이션을 위한 쿼리 파라미터 활용
GET /api/markets?status=active&sort=volume&limit=20&offset=0
```

### 리포지토리 패턴 (Repository Pattern)

데이터 액세스 로직을 추상화하여 비즈니스 로직과 분리합니다.

```typescript
// 데이터 액세스 로직 추상화
interface MarketRepository {
  findAll(filters?: MarketFilters): Promise<Market[]>
  findById(id: string): Promise<Market | null>
  create(data: CreateMarketDto): Promise<Market>
  update(id: string, data: UpdateMarketDto): Promise<Market>
  delete(id: string): Promise<void>
}

class SupabaseMarketRepository implements MarketRepository {
  async findAll(filters?: MarketFilters): Promise<Market[]> {
    let query = supabase.from('markets').select('*')

    if (filters?.status) {
      query = query.eq('status', filters.status)
    }

    if (filters?.limit) {
      query = query.limit(filters.limit)
    }

    const { data, error } = await query

    if (error) throw new Error(error.message)
    return data
  }

  // 기타 메서드 구현...
}
```

### 서비스 레이어 패턴 (Service Layer Pattern)

데이터 액세스와 분리된 비즈니스 로직을 담당합니다.

```typescript
// 데이터 액세스와 분리된 비즈니스 로직
class MarketService {
  constructor(private marketRepo: MarketRepository) {}

  async searchMarkets(query: string, limit: number = 10): Promise<Market[]> {
    // 비즈니스 로직 수행
    const embedding = await generateEmbedding(query)
    const results = await this.vectorSearch(embedding, limit)

    // 전체 데이터 조회
    const markets = await this.marketRepo.findByIds(results.map(r => r.id))

    // 유사도 순으로 정렬
    return markets.sort((a, b) => {
      const scoreA = results.find(r => r.id === a.id)?.score || 0
      const scoreB = results.find(r => r.id === b.id)?.score || 0
      return scoreA - scoreB
    })
  }

  private async vectorSearch(embedding: number[], limit: number) {
    // 벡터 검색 구현부
  }
}
```

### 미들웨어 패턴 (Middleware Pattern)

요청/응답 처리 파이프라인을 구축합니다.

```typescript
// 요청/응답 처리 파이프라인
export function withAuth(handler: NextApiHandler): NextApiHandler {
  return async (req, res) => {
    const token = req.headers.authorization?.replace('Bearer ', '')

    if (!token) {
      return res.status(401).json({ error: 'Unauthorized' })
    }

    try {
      const user = await verifyToken(token)
      req.user = user
      return handler(req, res)
    } catch (error) {
      return res.status(401).json({ error: 'Invalid token' })
    }
  }
}

// 사용 예시
export default withAuth(async (req, res) => {
  // 핸들러에서 req.user에 접근 가능
})
```

## 데이터베이스 패턴

### 쿼리 최적화

```typescript
// ✅ 좋음: 필요한 컬럼만 선택
const { data } = await supabase
  .from('markets')
  .select('id, name, status, volume')
  .eq('status', 'active')
  .order('volume', { ascending: false })
  .limit(10)

// ❌ 나쁨: 모든 컬럼(*) 선택
const { data } = await supabase
  .from('markets')
  .select('*')
```

### N+1 쿼리 방지

```typescript
// ❌ 나쁨: N+1 쿼리 문제 발생
const markets = await getMarkets()
for (const market of markets) {
  market.creator = await getUser(market.creator_id)  // N번의 추가 쿼리 발생
}

// ✅ 좋음: 배치(Batch) 조회 활용
const markets = await getMarkets()
const creatorIds = markets.map(m => m.creator_id)
const creators = await getUsers(creatorIds)  // 1번의 쿼리로 모든 사용자 조회
const creatorMap = new Map(creators.map(c => [c.id, c]))

markets.forEach(market => {
  market.creator = creatorMap.get(market.creator_id)
})
```

## 캐싱 전략

### Redis 캐싱 레이어

```typescript
class CachedMarketRepository implements MarketRepository {
  constructor(
    private baseRepo: MarketRepository,
    private redis: RedisClient
  ) {}

  async findById(id: string): Promise<Market | null> {
    // 1. 먼저 캐시 확인
    const cached = await this.redis.get(`market:${id}`)

    if (cached) {
      return JSON.parse(cached)
    }

    // 2. 캐시 미스 시 데이터베이스에서 조회
    const market = await this.baseRepo.findById(id)

    if (market) {
      // 3. 5분 동안 캐시 저장
      await this.redis.setex(`market:${id}`, 300, JSON.stringify(market))
    }

    return market
  }

  async invalidateCache(id: string): Promise<void> {
    await this.redis.del(`market:${id}`)
  }
}
```

## 에러 처리 패턴

### 중앙 집중식 에러 핸들러

```typescript
class ApiError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message)
    Object.setPrototypeOf(this, ApiError.prototype)
  }
}

export function errorHandler(error: unknown, req: Request): Response {
  if (error instanceof ApiError) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: error.statusCode })
  }

  if (error instanceof z.ZodError) {
    return NextResponse.json({
      success: false,
      error: 'Validation failed',
      details: error.errors
    }, { status: 400 })
  }

  // 예상치 못한 에러 로깅
  console.error('Unexpected error:', error)

  return NextResponse.json({
    success: false,
    error: 'Internal server error'
  }, { status: 500 })
}
```

## 백그라운드 작업 및 큐 (Background Jobs & Queues)

코드 실행을 블로킹하지 않고 작업을 큐에 추가하여 비동기로 처리합니다.

```typescript
class JobQueue<T> {
  private queue: T[] = []
  private processing = false

  async add(job: T): Promise<void> {
    this.queue.push(job)

    if (!this.processing) {
      this.process()
    }
  }

  private async process(): Promise<void> {
    this.processing = true

    while (this.queue.length > 0) {
      const job = this.queue.shift()!

      try {
        await this.execute(job)
      } catch (error) {
        console.error('Job failed:', error)
      }
    }

    this.processing = false
  }

  private async execute(job: T): Promise<void> {
    // 작업 실행 로직
  }
}
```

**기억하십시오**: 백엔드 패턴은 확장 가능하고 유지보수 가능한 서버측 애플리케이션을 가능하게 합니다. 프로젝트의 복잡도에 맞는 적절한 패턴을 선택하십시오.
    
