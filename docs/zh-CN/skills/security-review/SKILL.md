---
name: security-review
description: 인증 추가, 사용자 입력 처리, 비밀 정보 관리, API 엔드포인트 생성 또는 결제/민감 기능을 구현할 때 이 스킬을 사용하십시오. 종합적인 보안 체크리스트와 패턴을 제공합니다.
origin: ECC
---

# 보안 리뷰 스킬 (Security Review)

이 스킬은 모든 코드가 보안 베스트 프랙티스를 준수하도록 보장하고 잠재적인 취약점을 식별하는 데 도움을 줍니다.

## 적용 시점

* 인증(Authentication) 및 인가(Authorization)를 구현할 때
* 사용자 입력 처리 및 파일 업로드 기능을 만들 때
* 새로운 API 엔드포인트를 생성할 때
* API 키, 비밀번호 등 비밀 정보(Secrets)를 다룰 때
* 결제 기능을 구현할 때
* 민감한 데이터를 저장하거나 전송할 때
* 서드파티 API를 연동할 때

## 보안 체크리스트

### 1. 비밀 정보 관리 (Secrets Management)

#### ❌ 절대 하지 마십시오
```typescript
const apiKey = "sk-proj-xxxxx"  // 소스 코드 내 하드코딩된 API 키
const dbPassword = "password123" // 소스 코드 내 하드코딩된 DB 비밀번호
```

#### ✅ 항상 이렇게 하십시오
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 비밀 정보 존재 여부 확인
if (!apiKey) {
  throw new Error('OPENAI_API_KEY 환경 변수가 설정되지 않았습니다.')
}
```

**검증 단계:**
* [ ] 하드코딩된 API 키, 토큰, 비밀번호가 없는가?
* [ ] 모든 비밀 정보가 환경 변수(`.env`)로 관리되는가?
* [ ] `.env` 파일이 `.gitignore`에 포함되어 있는가?
* [ ] Git 이력(History)에 비밀 정보가 포함되어 있지 않은가?
* [ ] 운영 환경용 비밀 정보가 플랫폼(Vercel, Railway 등)에 안전하게 등록되어 있는가?

### 2. 입력값 검증 (Input Validation)

#### 사용자 입력은 반드시 검증하십시오
```typescript
import { z } from 'zod'

// 검증 스키마 정의
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 처리 전 검증 실행
export async function createUser(input: unknown) {
  const validated = CreateUserSchema.parse(input)
  return await db.users.create(validated)
}
```

#### 파일 업로드 검증
* **크기 제한**: 예) 최대 5MB
* **타입 제한**: 예) `image/jpeg`, `image/png` 등 허용된 MIME 타입만 체크
* **확장자 제한**: 화이트리스트 기반의 확장자 체크

### 3. SQL 인젝션 방지

#### ❌ 문자열 결합을 통한 SQL 생성 금지
```typescript
// 위험 - SQL 인젝션 취약점 발생
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
```

#### ✅ 파라미터화된 쿼리 사용
```typescript
// 안전 - 파라미터화된 방식
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// 또는 Raw SQL의 경우
await db.query('SELECT * FROM users WHERE email = $1', [userEmail])
```

### 4. 인증 및 인가 (Auth)

* **토큰 저장**: JWT 토큰 등은 XSS 취약점에 노출된 `localStorage` 대신 `httpOnly` 쿠키에 저장하십시오.
* **권한 체크**: 민감한 작업을 실행하기 전, 요청자의 역할(Admin 등)을 반드시 확인하십시오.
* **RLS (Supabase)**: 데이터베이스 수준에서 행 단위 보안(Row Level Security)을 활성화하십시오.

### 5. XSS 방지

사용자가 제공한 HTML을 렌더링해야 할 경우, 반드시 `DOMPurify` 등을 사용하여 새니타이징(Sanitize)하십시오.

```typescript
import DOMPurify from 'isomorphic-dompurify'

const clean = DOMPurify.sanitize(userContent)
```

### 6. CSRF 방지

* 상태 변경(POST, PUT, DELETE) 요청에는 CSRF 토큰을 활용하십시오.
* 모든 쿠키에 `SameSite=Strict` 설정을 적용하십시오.

### 7. 속도 제한 (Rate Limiting)

무차별 대입 공격(Brute-force)이나 DOS 공격을 방어하기 위해 API 엔드포인트에 속도 제한을 적용하십시오. 검색이나 결제 등 비용이 많이 드는 작업에는 더 엄격한 제한을 적용합니다.

### 8. 민감한 데이터 노출 방지

* **로그 관리**: 비밀번호, 토큰, 신용카드 번호 등이 로그에 기록되지 않도록 마스킹 처리하십시오.
* **에러 메시지**: 사용자에게는 일반적인 에러 메시지를 보여주고, 상세한 스택 트레이스나 내부 정보는 서버 로그에만 기록하십시오.

### 9. 블록체인 보안 (Solana 등)

* 지갑 서명 검증을 거쳐 소유권을 확인하십시오.
* 트랜잭션 전 잔액을 확인하고, 수신인 주소와 금액이 예상과 일치하는지 검증하십시오.

### 10. 의존성 보안

* 정기적으로 `npm audit`을 실행하여 취약점을 점검하십시오.
* `package-lock.json` 등 락 파일을 반드시 관리하고 CI/CD에서 활용하십시오.

---

## 배포 전 보안 체크리스트 (최종)

* [ ] **비밀 정보**: 하드코딩된 키가 없는가?
* [ ] **입력 검증**: 모든 사용자 입력이 스키마로 검증되는가?
* [ ] **SQL 인젝션**: 모든 쿼리가 파라미터화되어 있는가?
* [ ] **XSS**: 사용자 제공 콘텐츠가 새니타이징되는가?
* [ ] **인증/인가**: 토큰 처리가 안전하며 권한 체크가 이루어지는가?
* [ ] **속도 제한**: 주요 엔드포인트에 제한이 설정되어 있는가?
* [ ] **에러/로그**: 민감 정보 누출이 없는가?
* [ ] **의존성**: 최신 상태이며 보안 취약점이 해결되었는가?

**핵심**: 보안은 선택 사항이 아닙니다. 단 하나의 취약점이 전체 플랫폼을 위험에 빠뜨릴 수 있습니다. 의심스러운 경우 가장 안전한 방향으로 결정하십시오.
