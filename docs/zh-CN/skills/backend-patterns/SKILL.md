---
name: backend-patterns
description: 백엔드 아키텍처 패턴, API 설계, 데이터베이스 최적화와 Node.js, Express, Next.js API 라우트에 적용 가능한 서버 측 베스트 프랙티스입니다.
origin: ECC
---

# 백엔드 개발 패턴

확장 가능한 서버 측 애플리케이션을 위한 백엔드 아키텍처 패턴과 베스트 프랙티스입니다.

## 적용 시점

* REST 또는 GraphQL API 엔드포인트를 설계할 때
* 저장소(Repository), 서비스(Service), 컨트롤러(Controller) 레이어를 구현할 때
* 데이터베이스 쿼리를 최적화할 때 (N+1 문제, 인덱싱, 커넥션 풀링 등)
* 캐싱 로직을 추가할 때 (Redis, 인메모리 캐시, HTTP 캐시 헤더 등)
* 백그라운드 작업이나 비동기 처리를 설정할 때
* API 에러 처리 및 검증 구조를 구축할 때
* 미들웨어(인증, 로깅, 속도 제한 등)를 구현할 때

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

### 저장소 (Repository) 패턴

```typescript
// 데이터 접근 로직 추상화
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

### 서비스 (Service) 레이어 패턴

```typescript
// 데이터 접근과 분리된 비즈니스 로직
class MarketService {
  constructor(private marketRepo: MarketRepository) {}

  async searchMarkets(query: string, limit: number = 10): Promise<Market[]> {
    // 비즈니스 로직 처리
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
    // 벡터 검색 구현체
  }
}
```

### 미들웨어 (Middleware) 패턴

```typescript
// 요청/응답 처리 파이프라인
export function withAuth(handler: NextApiHandler): NextApiHandler {
  return async (req, res) => {
    const token = req.headers.authorization?.replace('Bearer ', '')

    if (!token) {
      return res.status(401).json({ error: '인증되지 않음(Unauthorized)' })
    }

    try {
      const user = await verifyToken(token)
      req.user = user
      return handler(req, res)
    } catch (error) {
      return res.status(401).json({ error: '유효하지 않은 토큰' })
    }
  }
}

// 사용 예시
export default withAuth(async (req, res) => {
  // 핸들러 내에서 req.user에 접근 가능
})
```

## 데이터베이스 패턴

### 쿼리 최적화

```typescript
// ✅ 좋은 예: 필요한 컬럼만 선택
const { data } = await supabase
  .from('markets')
  .select('id, name, status, volume')
  .eq('status', 'active')
  .order('volume', { ascending: false })
  .limit(10)

// ❌ 나쁜 예: 모든 컬럼 선택 (*)
const { data } = await supabase
  .from('markets')
  .select('*')
```

### N+1 쿼리 방지

```typescript
// ❌ 나쁜 예: N+1 쿼리 문제 발생
const markets = await getMarkets()
for (const market of markets) {
  market.creator = await getUser(market.creator_id)  // 루프 내에서 N번 쿼리 실행
}

// ✅ 좋은 예: 배치 처리(Batch Fetching)
const markets = await getMarkets()
const creatorIds = markets.map(m => m.creator_id)
const creators = await getUsers(creatorIds)  // 단 1번의 쿼리로 모든 데이터 조회
const creatorMap = new Map(creators.map(c => [c.id, c]))

markets.forEach(market => {
  market.creator = creatorMap.get(market.creator_id)
})
```

### 트랜잭션 (Transaction) 패턴

```typescript
async function createMarketWithPosition(
  marketData: CreateMarketDto,
  positionData: CreatePositionDto
) {
  // Supabase RPC(Stored Procedure)를 통한 트랜잭션 처리
  const { data, error } = await supabase.rpc('create_market_with_position', {
    market_data: marketData,
    position_data: positionData
  })

  if (error) throw new Error('트랜잭션 실패')
  return data
}

// Supabase 내 SQL 함수 정의 예시
CREATE OR REPLACE FUNCTION create_market_with_position(
  market_data jsonb,
  position_data jsonb
)
RETURNS jsonb
LANGUAGE plpgsql
AS $$
BEGIN
  -- 트랜잭션 자동 시작
  INSERT INTO markets VALUES (market_data);
  INSERT INTO positions VALUES (position_data);
  RETURN jsonb_build_object('success', true);
EXCEPTION
  WHEN OTHERS THEN
    -- 오류 발생 시 자동 롤백
    RETURN jsonb_build_object('success', false, 'error', SQLERRM);
END;
$$;
```

## 캐싱 전략

### Redis 캐시 레이어

```typescript
class CachedMarketRepository implements MarketRepository {
  constructor(
    private baseRepo: MarketRepository,
    private redis: RedisClient
  ) {}

  async findById(id: string): Promise<Market | null> {
    // 1. 캐시 먼저 확인
    const cached = await this.redis.get(`market:${id}`)

    if (cached) {
      return JSON.parse(cached)
    }

    // 2. 캐시 미스 시 데이터베이스에서 조회
    const market = await this.baseRepo.findById(id)

    if (market) {
      // 3. 5분 동안 캐싱 (300초)
      await this.redis.setex(`market:${id}`, 300, JSON.stringify(market))
    }

    return market
  }

  async invalidateCache(id: string): Promise<void> {
    await this.redis.del(`market:${id}`)
  }
}
```

### 캐시-어사이드 (Cache-Aside) 패턴

```typescript
async function getMarketWithCache(id: string): Promise<Market> {
  const cacheKey = `market:${id}`

  // 캐시 확인
  const cached = await redis.get(cacheKey)
  if (cached) return JSON.parse(cached)

  // 캐시 미스 시 DB 조회
  const market = await db.markets.findUnique({ where: { id } })

  if (!market) throw new Error('Market을 찾을 수 없습니다')

  // 캐시 업데이트
  await redis.setex(cacheKey, 300, JSON.stringify(market))

  return market
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
      error: '검증 실패(Validation failed)',
      details: error.errors
    }, { status: 400 })
  }

  // 예기치 않은 에러 로깅
  console.error('예기치 않은 에러:', error)

  return NextResponse.json({
    success: false,
    error: '내부 서버 오류(Internal server error)'
  }, { status: 500 })
}

// API 라우트에서 사용 예시
export async function GET(request: Request) {
  try {
    const data = await fetchData()
    return NextResponse.json({ success: true, data })
  } catch (error) {
    return errorHandler(error, request)
  }
}
```

### 지수 백오프(Exponential Backoff) 재시도

```typescript
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  let lastError: Error

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn()
    } catch (error) {
      lastError = error as Error

      if (i < maxRetries - 1) {
        // 지수 백오프 적용: 1초, 2초, 4초 대기
        const delay = Math.pow(2, i) * 1000
        await new Promise(resolve => setTimeout(resolve, delay))
      }
    }
  }

  throw lastError!
}

// 사용 예시
const data = await fetchWithRetry(() => fetchFromAPI())
```

## 인증 및 권한 부여

### JWT 토큰 검증

```typescript
import jwt from 'jsonwebtoken'

interface JWTPayload {
  userId: string
  email: string
  role: 'admin' | 'user'
}

export function verifyToken(token: string): JWTPayload {
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload
    return payload
  } catch (error) {
    throw new ApiError(401, '유효하지 않은 토큰')
  }
}

export async function requireAuth(request: Request) {
  const token = request.headers.get('authorization')?.replace('Bearer ', '')

  if (!token) {
    throw new ApiError(401, '인증 토큰이 누락되었습니다')
  }

  return verifyToken(token)
}

// API 라우트 사용 예시
export async function GET(request: Request) {
  const user = await requireAuth(request)

  const data = await getDataForUser(user.userId)

  return NextResponse.json({ success: true, data })
}
```

### 역할 기반 접근 제어 (RBAC)

```typescript
type Permission = 'read' | 'write' | 'delete' | 'admin'

interface User {
  id: string
  role: 'admin' | 'moderator' | 'user'
}

const rolePermissions: Record<User['role'], Permission[]> = {
  admin: ['read', 'write', 'delete', 'admin'],
  moderator: ['read', 'write', 'delete'],
  user: ['read', 'write']
}

export function hasPermission(user: User, permission: Permission): boolean {
  return rolePermissions[user.role].includes(permission)
}

export function requirePermission(permission: Permission) {
  return (handler: (request: Request, user: User) => Promise<Response>) => {
    return async (request: Request) => {
      const user = await requireAuth(request)

      if (!hasPermission(user, permission)) {
        throw new ApiError(403, '권한이 부족합니다')
      }

      return handler(request, user)
    }
  }
}

// 고차 함수(HOF)로 핸들러를 래핑하여 사용
export const DELETE = requirePermission('delete')(
  async (request: Request, user: User) => {
    // 핸들러는 인증되고 권한이 검증된 사용자를 인자로 받음
    return new Response('삭제 완료', { status: 200 })
  }
)
```

## 속도 제한 (Rate Limiting)

### 단순 인메모리 속도 제한기

```typescript
class RateLimiter {
  private requests = new Map<string, number[]>()

  async checkLimit(
    identifier: string,
    maxRequests: number,
    windowMs: number
  ): Promise<boolean> {
    const now = Date.now()
    const requests = this.requests.get(identifier) || []

    // 윈도우 범위를 벗어난 오래된 요청 제거
    const recentRequests = requests.filter(time => now - time < windowMs)

    if (recentRequests.length >= maxRequests) {
      return false  // 제한 초과
    }

    // 현재 요청 추가
    recentRequests.push(now)
    this.requests.set(identifier, recentRequests)

    return true
  }
}

const limiter = new RateLimiter()

export async function GET(request: Request) {
  const ip = request.headers.get('x-forwarded-for') || 'unknown'

  const allowed = await limiter.checkLimit(ip, 100, 60000)  // 분당 100회 제한

  if (!allowed) {
    return NextResponse.json({
      error: '속도 제한을 초과했습니다'
    }, { status: 429 })
  }

  // 요청 처리 계속 진행...
}
```

## 백그라운드 작업 및 큐 (Queue)

### 단순 큐 패턴

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
        console.error('작업 수행 실패:', error)
      }
    }

    this.processing = false
  }

  private async execute(job: T): Promise<void> {
    // 작업 실행 로직
  }
}

// 예시: 마켓 정보 인덱싱 큐
interface IndexJob {
  marketId: string
}

const indexQueue = new JobQueue<IndexJob>()

export async function POST(request: Request) {
  const { marketId } = await request.json()

  // 요청을 차단하지 않고 큐에 추가
  await indexQueue.add({ marketId })

  return NextResponse.json({ success: true, message: '작업이 큐에 추가되었습니다' })
}
```

## 로깅 및 모니터링

### 구조화된 로깅 (Structured Logging)

```typescript
interface LogContext {
  userId?: string
  requestId?: string
  method?: string
  path?: string
  [key: string]: unknown
}

class Logger {
  log(level: 'info' | 'warn' | 'error', message: string, context?: LogContext) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      ...context
    }

    console.log(JSON.stringify(entry))
  }

  info(message: string, context?: LogContext) {
    this.log('info', message, context)
  }

  warn(message: string, context?: LogContext) {
    this.log('warn', message, context)
  }

  error(message: string, error: Error, context?: LogContext) {
    this.log('error', message, {
      ...context,
      error: error.message,
      stack: error.stack
    })
  }
}

const logger = new Logger()

// 사용 예시
export async function GET(request: Request) {
  const requestId = crypto.randomUUID()

  logger.info('마켓 정보 조회 시작', {
    requestId,
    method: 'GET',
    path: '/api/markets'
  })

  try {
    const markets = await fetchMarkets()
    return NextResponse.json({ success: true, data: markets })
  } catch (error) {
    logger.error('마켓 정보 조회 실패', error as Error, { requestId })
    return NextResponse.json({ error: '내부 에러' }, { status: 500 })
  }
}
```

**주의**: 백엔드 패턴은 확장 가능하고 유지보수가 용이한 서버 측 애플리케이션 개발을 돕습니다. 현재 프로젝트의 복잡도와 규모에 적합한 패턴을 선택하십시오.
