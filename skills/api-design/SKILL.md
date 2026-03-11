---
name: api-design
description: 리소스 명명, 상태 코드, 페이지네이션, 필터링, 에러 응답, 버전 관리 및 속도 제한을 포함한 운영용 REST API 설계 패턴입니다.
origin: ECC
---

# API 설계 패턴 (API Design Patterns)

일관되고 개발자 친화적인 REST API를 설계하기 위한 컨벤션과 최선 관행(Best practices)을 안내합니다.

## 활성화 시점

- 새로운 API 엔드포인트 설계 시
- 기존 API 규약(Contract) 검토 시
- 페이지네이션, 필터링 또는 정렬 기능 추가 시
- API 에러 처리 구현 시
- API 버전 관리 전략 수립 시
- 공개용 또는 파트너용 API 구축 시

## 리소스 설계

### URL 구조

```
# 리소스는 명사, 복수형, 소문자, 케밥 케이스(kebab-case)를 사용합니다.
GET    /api/v1/users
GET    /api/v1/users/:id
POST   /api/v1/users
PUT    /api/v1/users/:id
PATCH  /api/v1/users/:id
DELETE /api/v1/users/:id

# 관계를 나타내는 하위 리소스
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# CRUD로 매핑되지 않는 동작 (동사는 가급적 제한적으로 사용)
POST   /api/v1/orders/:id/cancel
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
```

### 명명 규칙

```
# 좋음 (GOOD)
/api/v1/team-members          # 여러 단어로 된 리소스는 케밥 케이스 사용
/api/v1/orders?status=active  # 필터링을 위해 쿼리 파라미터 사용
/api/v1/users/123/orders      # 소유 관계를 위해 중첩 리소스 사용

# 나쁨 (BAD)
/api/v1/getUsers              # URL에 동사 포함
/api/v1/user                  # 단수형 사용 (복수형 권장)
/api/v1/team_members          # URL에 스네이크 케이스(snake_case) 사용
/api/v1/users/123/getOrders   # 하위 리소스에 동사 포함
```

## HTTP 메서드 및 상태 코드

### 메서드 의미론 (Method Semantics)

| 메서드 | 멱등성 (Idempotent) | 안전함 (Safe) | 용도 |
|--------|-----------|------|---------|
| GET | 예 | 예 | 리소스 조회 |
| POST | 아니오 | 아니오 | 리소스 생성, 동작 트리거 |
| PUT | 예 | 아니오 | 리소스 전체 교체 |
| PATCH | 아니오* | 아니오 | 리소스의 일부 업데이트 |
| DELETE | 예 | 아니오 | 리소스 삭제 |

*PATCH는 적절한 구현을 통해 멱등성을 보장받을 수 있습니다.

### 상태 코드 참조

```
# 성공 (Success)
200 OK                    — GET, PUT, PATCH (응답 바디가 있는 경우)
201 Created               — POST (Location 헤더 포함 권장)
204 No Content            — DELETE, PUT (응답 바디가 없는 경우)

# 클라이언트 에러 (Client Errors)
400 Bad Request           — 유효성 검사 실패, 잘못된 JSON 형식
401 Unauthorized          — 인증 누락 또는 잘못된 인증
403 Forbidden             — 인증되었으나 권한이 없음
404 Not Found             — 리소스가 존재하지 않음
409 Conflict              — 중복 항목, 상태 충돌
422 Unprocessable Entity  — 의미론적으로 부적절함 (JSON은 올바르나 데이터가 잘못됨)
429 Too Many Requests     — 속도 제한(Rate limit) 초과

# 서버 에러 (Server Errors)
500 Internal Server Error — 예상치 못한 실패 (상세 내용은 절대 노출하지 말 것)
502 Bad Gateway           — 상위 서비스 실패
503 Service Unavailable   — 일시적인 과부하, Retry-After 포함 권장
```

### 흔한 실수

```
# 나쁨 (BAD): 모든 상황에 200 반환
{ "status": 200, "success": false, "error": "Not found" }

# 좋음 (GOOD): HTTP 상태 코드를 의미에 맞게 사용
HTTP/1.1 404 Not Found
{ "error": { "code": "not_found", "message": "User not found" } }

# 나쁨 (BAD): 유효성 검사 에러에 500 반환
# 좋음 (GOOD): 필드 수준의 상세 정보를 포함하여 400 또는 422 반환

# 나쁨 (BAD): 리소스 생성 시 200 반환
# 좋음 (GOOD): Location 헤더와 함께 201 반환
HTTP/1.1 201 Created
Location: /api/v1/users/abc-123
```

## 응답 형식

### 성공 응답

```json
{
  "data": {
    "id": "abc-123",
    "email": "alice@example.com",
    "name": "Alice",
    "created_at": "2025-01-15T10:30:00Z"
  }
}
```

### 컬렉션 응답 (페이지네이션 포함)

```json
{
  "data": [
    { "id": "abc-123", "name": "Alice" },
    { "id": "def-456", "name": "Bob" }
  ],
  "meta": {
    "total": 142,
    "page": 1,
    "per_page": 20,
    "total_pages": 8
  },
  "links": {
    "self": "/api/v1/users?page=1&per_page=20",
    "next": "/api/v1/users?page=2&per_page=20",
    "last": "/api/v1/users?page=8&per_page=20"
  }
}
```

### 에러 응답

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "code": "invalid_format"
      },
      {
        "field": "age",
        "message": "Must be between 0 and 150",
        "code": "out_of_range"
      }
    ]
  }
}
```

## 페이지네이션 (Pagination)

### 오프셋 기반 (Offset-Based, 단순형)

```
GET /api/v1/users?page=2&per_page=20

# 구현 예시
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 20 OFFSET 20;
```

**장점:** 구현이 쉬움, "N 페이지로 이동" 기능 지원 가능
**단점:** 오프셋이 커질수록 성능 저하 발생, 동시 삽입 시 데이터 중복/누락 가능성

### 커서 기반 (Cursor-Based, 확장형)

```
GET /api/v1/users?cursor=eyJpZCI6MTIzfQ&limit=20

# 구현 예시
SELECT * FROM users
WHERE id > :cursor_id
ORDER BY id ASC
LIMIT 21;  -- has_next 여부를 판단하기 위해 하나 더 가져옴
```

**장점:** 위치에 상관없이 일관된 성능 유지, 동시 삽입 상황에서도 안정적
**단점:** 임의의 페이지로 바로 점프할 수 없음, 커서 값이 불투명함

### 활용 가이드

| 유스케이스 | 페이지네이션 유형 |
|----------|----------------|
| 관리자 대시보드, 소규모 데이터셋 (<1만 건) | 오프셋(Offset) |
| 무한 스크롤, 피드, 대규모 데이터셋 | 커서(Cursor) |
| 공개용 API | 커서(Cursor) 기본, 오프셋(Offset) 선택 사항 |
| 검색 결과 | 오프셋(Offset, 페이지 번호 필요) |

## 필터링, 정렬 및 검색

### 필터링 (Filtering)

```
# 단순 동등 비교
GET /api/v1/orders?status=active&customer_id=abc-123

# 비교 연산자 (대괄호 표기법 사용)
GET /api/v1/products?price[gte]=10&price[lte]=100
GET /api/v1/orders?created_at[after]=2025-01-01

# 다중 값 (쉼표 구분)
GET /api/v1/products?category=electronics,clothing

# 중첩 필드 (점 표기법)
GET /api/v1/orders?customer.country=US
```

### 정렬 (Sorting)

```
# 단일 필드 (내림차순은 접두사 - 사용)
GET /api/v1/products?sort=-created_at

# 다중 필드 (쉼표 구분)
GET /api/v1/products?sort=-featured,price,-created_at
```

### 전체 텍스트 검색 (Full-Text Search)

```
# 검색 쿼리 파라미터
GET /api/v1/products?q=wireless+headphones

# 특정 필드 검색
GET /api/v1/users?email=alice
```

## 인증 및 인가 (Authentication/Authorization)

### 토큰 기반 인증

```
# Authorization 헤더에 Bearer 토큰 포함
GET /api/v1/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# API 키 (서버 간 통신용)
GET /api/v1/data
X-API-Key: sk_live_abc123
```

## 속도 제한 (Rate Limiting)

### 응답 헤더

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000

# 제한 초과 시
HTTP/1.1 429 Too Many Requests
Retry-After: 60
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Try again in 60 seconds."
  }
}
```

## 버전 관리 (Versioning)

### URL 경로 버전 관리 (권장)

```
/api/v1/users
/api/v2/users
```

**장점:** 명시적임, 라우팅이 쉬움, 캐싱 가능
**단점:** 버전 간에 URL이 변경됨

### 버전 관리 전략

1. `/api/v1/`로 시작하십시오 — 꼭 필요한 경우가 아니면 버전을 올리지 마십시오.
2. 최대 2개의 활성 버전만 유지하십시오 (현재 버전 + 이전 버전).
3. 일몰(Deprecation) 타임라인 계획:
   - 일몰 예고 (공개 API의 경우 최소 6개월 전 공지)
   - Sunset 헤더 추가: `Sunset: Sat, 01 Jan 2026 00:00:00 GMT`
   - 일몰 후에는 410 Gone 반환
4. 하위 호환성이 유지되는 변경 사항은 새로운 버전이 필요하지 않습니다:
   - 응답에 새로운 필드 추가
   - 새로운 옵셔널 쿼리 파라미터 추가
   - 새로운 엔드포인트 추가
5. 하위 호환성이 깨지는 변경 사항은 새로운 버전이 필요합니다:
   - 기존 필드 삭제 또는 이름 변경
   - 필드 타입 변경
   - URL 구조 변경
   - 인증 방식 변경

## API 설계 체크리스트

새로운 엔드포인트를 출시하기 전에 확인하십시오:

- [ ] 리소스 URL이 명명 규칙(복수형, 케밥 케이스, 동사 배제)을 따르는가
- [ ] 올바른 HTTP 메서드를 사용하고 있는가 (조회는 GET, 생성은 POST 등)
- [ ] 적절한 상태 코드를 반환하는가 (모든 경우에 200 반환 지양)
- [ ] 입력값이 스키마(Zod, Pydantic 등)에 의해 검증되는가
- [ ] 에러 응답이 코드와 메시지를 포함한 표준 형식을 따르는가
- [ ] 리스트 엔드포인트에 페이지네이션(커서 또는 오프셋)이 구현되어 있는가
- [ ] 인증이 요구되는가 (또는 명시적으로 공개 엔드포인트로 설정되었는가)
- [ ] 인가가 확인되는가 (사용자가 자신의 리소스에만 접근 가능한가)
- [ ] 속도 제한(Rate limit)이 구성되어 있는가
- [ ] 응답에 내부 상세 정보(스택 트레이스, SQL 에러 등)가 유출되지 않는가
- [ ] 기존 엔드포인트와 일관된 명명 규칙(camelCase vs snake_case)을 따르는가
- [ ] 문서화(OpenAPI/Swagger 등)가 업데이트되었는가
    
