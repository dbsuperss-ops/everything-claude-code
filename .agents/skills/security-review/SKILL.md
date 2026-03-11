---
name: security-review
description: 인증 추가, 사용자 입력 처리, 시크릿(Secrets) 작업, API 엔드포인트 생성 또는 결제/민감한 기능 구현 시 이 스킬을 사용하십시오. 포괄적인 보안 체크리스트와 패턴을 제공합니다.
origin: ECC
---

# 보안 검토 스킬 (Security Review Skill)

이 스킬은 모든 코드가 보안 모범 사례를 따르도록 보장하며 잠재적인 취약점을 식별합니다.

## 활성화 시기

- 인증(Authentication) 또는 권한 부여(Authorization)를 구현할 때
- 사용자 입력 또는 파일 업로드를 처리할 때
- 새로운 API 엔드포인트를 생성할 때
- 시크릿이나 자격 증명(Credentials)을 다룰 때
- 결제 기능을 구현할 때
- 민감한 데이터를 저장하거나 전송할 때
- 제3자 API를 통합할 때

## 보안 체크리스트

### 1. 시크릿 관리 (Secrets Management)

#### ❌ 절대 하지 마십시오
```typescript
const apiKey = "sk-proj-xxxxx"  // 하드코딩된 시크릿
const dbPassword = "password123" // 소스 코드 내에 포함된 비밀번호
```

#### ✅ 항상 이렇게 하십시오
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 시크릿 존재 여부 확인
if (!apiKey) {
  throw new Error('OPENAI_API_KEY가 설정되지 않았습니다')
}
```

#### 점검 사항
- [ ] 하드코딩된 API 키, 토큰 또는 비밀번호가 없는가
- [ ] 모든 시크릿이 환경 변수로 관리되고 있는가
- [ ] `.env.local` 등 환경 변수 파일이 .gitignore에 포함되어 있는가
- [ ] Git 이력에 시크릿이 포함되어 있지 않은가
- [ ] 운영 환경 시크릿이 호스팅 플랫폼(Vercel, Railway 등)에 설정되었는가

### 2. 입력값 검증 (Input Validation)

#### 항상 사용자 입력을 검증하십시오
```typescript
import { z } from 'zod'

// 유효성 검사 스키마 정의
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 처리 전 검증 수행
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### 파일 업로드 검증
```typescript
function validateFileUpload(file: File) {
  // 크기 확인 (최대 5MB)
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('파일이 너무 큽니다 (최대 5MB)')
  }

  // 타입 확인
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('허용되지 않는 파일 형식입니다')
  }

  // 확장자 확인
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('허용되지 않는 파일 확장자입니다')
  }

  return true
}
```

#### 점검 사항
- [ ] 모든 사용자 입력이 스키마를 통해 검증되고 있는가
- [ ] 파일 업로드가 제한(크기, 타입, 확장자)되어 있는가
- [ ] 쿼리에 사용자 입력을 직접 사용하지 않는가
- [ ] 화이트리스트(Whitelist) 기반 검증 방식을 사용하는가
- [ ] 에러 메시지에 민감한 정보가 노출되지 않는가

### 3. SQL 인젝션 방지

#### ❌ 절대 SQL 문자열을 직접 연결하지 마십시오
```typescript
// 위험 - SQL 인젝션 취약점 발생 가능
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### ✅ 항상 매개변수화된 쿼리(Parameterized Queries)를 사용하십시오
```typescript
// 안전 - 매개변수화된 쿼리 사용
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// 또는 Raw SQL 사용 시
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 점검 사항
- [ ] 모든 데이터베이스 쿼리가 매개변수화된 방식을 사용하는가
- [ ] SQL에 문자열 연결 방식을 사용하지 않는가
- [ ] ORM 또는 쿼리 빌더를 올바르게 사용하고 있는가
- [ ] Supabase 쿼리가 제대로 정화(Sanitize)되었는가

### 4. 인증 및 권한 부여

#### JWT 토큰 처리
```typescript
// ❌ 잘못된 예: localStorage 사용 (XSS에 취약함)
localStorage.setItem('token', token)

// ✅ 올바른 예: httpOnly 쿠키 사용
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 권한 확인 (Authorization Checks)
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 항상 권한을 먼저 확인하십시오
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: '권한이 없습니다' },
      { status: 403 }
    )
  }

  // 삭제 진행
  await db.users.delete({ where: { id: userId } })
}
```

#### 행 레벨 보안 (Row Level Security, Supabase)
```sql
-- 모든 테이블에 RLS 활성화
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- 사용자는 자신의 데이터만 조회 가능
CREATE POLICY "사용자 본인 데이터 조회"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- 사용자는 자신의 데이터만 업데이트 가능
CREATE POLICY "사용자 본인 데이터 업데이트"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### 점검 사항
- [ ] 토큰이 localStorage가 아닌 httpOnly 쿠키에 저장되는가
- [ ] 민감한 작업 전에 권한 확인이 이루어지는가
- [ ] Supabase에서 행 레벨 보안(RLS)이 활성화되었는가
- [ ] 역할 기반 액세스 제어(RBAC)가 구현되었는가
- [ ] 세션 관리가 안전하게 이루어지는가

### 5. XSS 방지

#### HTML 정화 (Sanitize)
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 사용자로부터 제공받은 HTML은 항상 정화 처리하십시오
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### 콘텐츠 보안 정책 (CSP)
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

#### 점검 사항
- [ ] 사용자 제공 HTML이 정화되는가
- [ ] CSP 헤더가 설정되었는가
- [ ] 검증되지 않은 동적 콘텐츠 렌더링이 없는가
- [ ] React의 내장 XSS 보호 기능이 활용되고 있는가

### 6. CSRF 방지

#### CSRF 토큰
```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: '유효하지 않은 CSRF 토큰입니다' },
      { status: 403 }
    )
  }

  // 요청 처리
}
```

#### SameSite 쿠키
```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### 점검 사항
- [ ] 상태를 변경하는 작업에 CSRF 토큰이 사용되는가
- [ ] 모든 쿠키에 SameSite=Strict 속성이 설정되었는가
- [ ] Double-submit 쿠키 패턴이 구현되었는가

### 7. 속도 제한 (Rate Limiting)

#### API 속도 제한
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100, // 창 하나당 100회 요청 제한
  message: '요청이 너무 많습니다'
})

// 라우트에 적용
app.use('/api/', limiter)
```

#### 비용이 큰 작업
```typescript
// 검색 작업에 대해 더 엄격한 속도 제한 적용
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1분
  max: 10, // 분당 10회 요청 제한
  message: '검색 요청이 너무 많습니다'
})

app.use('/api/search', searchLimiter)
```

#### 점검 사항
- [ ] 모든 API 엔드포인트에 속도 제한이 적용되었는가
- [ ] 비용이 큰 작업에 대해 더 엄격한 제한이 있는가
- [ ] IP 기반 속도 제한이 이루어지는가
- [ ] 사용자별(인증 기준) 속도 제한이 이루어지는가

### 8. 민감한 데이터 노출 방지

#### 로깅 (Logging)
```typescript
// ❌ 잘못된 예: 민감한 데이터 로깅
console.log('사용자 로그인:', { email, password })
console.log('결제 정보:', { cardNumber, cvv })

// ✅ 올바른 예: 민감한 정보 마스킹
console.log('사용자 로그인:', { email, userId })
console.log('결제 정보:', { last4: card.last4, userId })
```

#### 에러 메시지
```typescript
// ❌ 잘못된 예: 내부 상세 정보 노출
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ 올바른 예: 일반적인 에러 메시지 반환
catch (error) {
  console.error('내부 오류 발생:', error)
  return NextResponse.json(
    { error: '오류가 발생했습니다. 다시 시도해 주세요.' },
    { status: 500 }
  )
}
```

#### 점검 사항
- [ ] 로그에 비밀번호, 토큰, 시크릿이 포함되지 않는가
- [ ] 사용자에게는 일반적인 에러 메시지만 표시되는가
- [ ] 상세 에러는 서버 로그에만 기록되는가
- [ ] 사용자에게 스택 트레이스가 노출되지 않는가

### 9. 블록체인 보안 (Solana)

#### 지갑 소유권 검증
```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

#### 트랜잭션 검증
```typescript
async function verifyTransaction(transaction: Transaction) {
  // 수신자 확인
  if (transaction.to !== expectedRecipient) {
    throw new Error('수신자가 올바르지 않습니다')
  }

  // 금액 확인
  if (transaction.amount > maxAmount) {
    throw new Error('금액이 한도를 초과했습니다')
  }

  // 잔액 확인
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('잔액이 부족합니다')
  }

  return true
}
```

#### 점검 사항
- [ ] 지갑 서명이 검증되는가
- [ ] 트랜잭션 상세 내용이 유효성 검사를 거치는가
- [ ] 트랜잭션 전 잔액 확인이 수행되는가
- [ ] 무작정(Blind) 트랜잭션 서명을 하지 않는가

### 10. 종속성(Dependency) 보안

#### 정기적인 업데이트
```bash
# 취약점 점검
npm audit

# 자동 수정 가능한 문제 해결
npm audit fix

# 종속성 업데이트
npm update

# 오래된 패키지 확인
npm outdated
```

#### 잠금 파일 (Lock Files)
```bash
# 잠금 파일은 항상 커밋하십시오
git add package-lock.json

# 재현 가능한 빌드를 위해 CI/CD에서 사용
npm ci  # npm install 대신 사용
```

#### 점검 사항
- [ ] 종속성이 최신 상태를 유지하고 있는가
- [ ] 알려진 취약점이 없는가 (npm audit clean)
- [ ] 잠금 파일이 커밋되었인가
- [ ] GitHub의 Dependabot이 활성화되었는가
- [ ] 정기적인 보안 업데이트가 이루어지는가

## 보안 테스트

### 자동화된 보안 테스트
```typescript
// 인증 테스트
test('인증이 필요함', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// 권한 부여 테스트
test('관리자 권한이 필요함', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// 입력값 검증 테스트
test('잘못된 입력을 거부함', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// 속도 제한 테스트
test('속도 제한을 적용함', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )

  const responses = await Promise.all(requests)
  const tooManyRequests = responses.filter(r => r.status === 429)

  expect(tooManyRequests.length).toBeGreaterThan(0)
})
```

## 배포 전 최종 보안 체크리스트

운영 환경 배포 전 필수 확인 사항:

- [ ] **시크릿**: 하드코딩된 시크릿이 없으며 모두 환경 변수로 관리됨
- [ ] **입력 검증**: 모든 사용자 입력이 검증됨
- [ ] **SQL 인젝션**: 모든 쿼리가 매개변수화됨
- [ ] **XSS**: 사용자 콘텐츠가 정화 처리됨
- [ ] **CSRF**: 보호 기능이 활성화됨
- [ ] **인증**: 적절한 토큰 처리 방식 사용
- [ ] **권한 부여**: 역할 확인 로직이 적용됨
- [ ] **속도 제한**: 모든 엔드포인트에 적용됨
- [ ] **HTTPS**: 운영 환경에서 필수 적용
- [ ] **보안 헤더**: CSP, X-Frame-Options 등이 설정됨
- [ ] **에러 처리**: 에러에 민감한 정보가 포함되지 않음
- [ ] **로깅**: 민감한 정보가 기록되지 않음
- [ ] **종속성**: 최신 상태이며 취약점 없음
- [ ] **행 레벨 보안**: Supabase에서 활성화됨
- [ ] **CORS**: 올바르게 구성됨
- [ ] **파일 업로드**: 검증 완료 (크기, 타입 등)
- [ ] **지갑 서명**: 검증 완료 (블록체인 사용 시)

## 리소스

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.js Security](https://nextjs.org/docs/security)
- [Supabase Security](https://supabase.com/docs/guides/auth)
- [Web Security Academy](https://portswigger.net/web-security)

---

**기억해 두세요**: 보안은 선택 사항이 아닙니다. 단 하나의 취약점이 플랫폼 전체를 위험에 빠뜨릴 수 있습니다. 의심스러울 때는 항상 더 안전한 방향을 선택하십시오.
