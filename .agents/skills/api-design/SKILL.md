---
name: api-design
description: 리소스 명칭 부여, 상태 코드, 페이지네이션, 필터링, 에러 응답, 버전 관리 및 운영용 API를 위한 속도 제한을 포함한 REST API 설계 패턴입니다.
origin: ECC
---

# API 설계 패턴 (API Design Patterns)

일관성 있고 개발자 친화적인 REST API 설계를 위한 관례 및 모범 사례입니다.

## 활성화 시기

- 새로운 API 엔드포인트를 설계할 때
- 기존 API 규약(Contract)을 검토할 때
- 페이지네이션, 필터링 또는 정렬 기능을 추가할 때
- API 에러 처리 로직을 구현할 때
- API 버전 관리 전략을 계획할 때
- 공용 또는 파트너 대상 API를 구축할 때

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

# 관계를 나타내기 위한 하위 리소스
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# CRUD에 매핑되지 않는 액션 (동사는 가급적 제한적으로 사용)
POST   /api/v1/orders/:id/cancel
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
```

### 명명 규칙

```
# 좋은 예
/api/v1/team-members          # 여러 단어로 된 리소스에는 케밥 케이스 사용
/api/v1/orders?status=active  # 필터링을 위해 쿼리 파라미터 사용
/api/v1/users/123/orders      # 소유 관계를 위해 중첩된 리소스 사용

# 나쁜 예
/api/v1/getUsers              # URL에 동사 포함
/api/v1/user                  # 단수형 (복수형 권장)
/api/v1/team_members          # URL에 스네이크 케이스(snake_case) 사용
/api/v1/users/123/getOrders   # 중첩 리소스에 동사 포함
```

## HTTP 메서드 및 상태 코드

### 메서드 의미론 (Method Semantics)

| 메서드 | 멱등성(Idempotent) | 안전성(Safe) | 용도 |
|--------|-----------|------|---------|
| GET | 예 | 예 | 리소스 조회 |
| POST | 아니요 | 아니요 | 리소스 생성, 액션 트리거 |
| PUT | 예 | 아니요 | 리소스 전체 교체 |
| PATCH | 아니요* | 아니요 | 리소스 일부 수정 |
| DELETE | 예 | 아니요 | 리소스 삭제 |

*PATCH는 구현 방식에 따라 멱등성을 갖도록 만들 수 있습니다.

### 상태 코드 참조

```
# 성공 (Success)
200 OK                    — GET, PUT, PATCH (응답 본문 포함 시)
201 Created               — POST (Location 헤더 포함 권장)
204 No Content            — DELETE, PUT (응답 본문 없을 때)

# 클라이언트 에러 (Client Errors)
400 Bad Request           — 유효성 검사 실패, 잘못된 형식의 JSON
401 Unauthorized          — 인증 정보 누락 또는 유효하지 않음
403 Forbidden             — 인증되었으나 권한이 없음
404 Not Found             — 리소스가 존재하지 않음
409 Conflict              — 중복 항목 발생, 상태 충돌
422 Unprocessable Entity  — 의미상 유효하지 않음 (JSON 형식은 맞으나 데이터가 부적절함)
429 Too Many Requests     — 속도 제한 초과

# 서버 에러 (Server Errors)
500 Internal Server Error — 예기치 않은 서버 오류 (상세 내용은 절대 노출하지 말 것)
502 Bad Gateway           — 상위 서버 에러
503 Service Unavailable   — 일시적인 과부하, Retry-After 헤더 포함 권장
```

### 흔히 하는 실수

```
# 나쁜 예: 모든 응답에 200 사용
{ "status": 200, "success": false, "error": "Not found" }

# 좋은 예: HTTP 상태 코드를 의미에 맞게 사용
HTTP/1.1 404 Not Found
{ "error": { "code": "not_found", "message": "사용자를 찾을 수 없습니다" } }

# 나쁜 예: 유효성 검사 에러에 500 사용
# 좋은 예: 400 또는 422를 사용하고 필드 단위 상세 정보를 제공

# 나쁜 예: 리소스 생성 시 200 사용
# 좋은 예: 201과 함께 Location 헤더 제공
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

### 목록 응답 (페이지네이션 포함)

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
    "message": "요청 유효성 검사 실패",
    "details": [
      {
        "field": "email",
        "message": "유효한 이메일 주소여야 합니다",
        "code": "invalid_format"
      },
      {
        "field": "age",
        "message": "0에서 150 사이의 숫자여야 합니다",
        "code": "out_of_range"
      }
    ]
  }
}
```

### 응답 봉투(Envelope) 변동 사항

```typescript
// 옵션 A: 데이터 래퍼가 포함된 봉투 방식 (공용 API 권장)
interface ApiResponse<T> {
  data: T;
  meta?: PaginationMeta;
  links?: PaginationLinks;
}

interface ApiError {
  error: {
    code: string;
    message: string;
    details?: FieldError[];
  };
}

// 옵션 B: 직접 반환 방식 (더 단순하며 내부 API용으로 선호됨)
// 성공 시: 리소스를 직접 반환
// 에러 시: 에러 객체 반환
// HTTP 상태 코드로 구분
```

## 페이지네이션 (Pagination)

### 오프셋 기반 (Offset-Based, 단순 방식)

```
GET /api/v1/users?page=2&per_page=20

# 구현 예시
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 20 OFFSET 20;
```

**장점:** 구현이 쉽고, "N 페이지로 이동" 기능을 지원함
**단점:** 높은 오프셋(예: OFFSET 100000)에서 속도가 느려지며, 데이터가 자주 추가되는 환경에서 중복 노출 가능성 있음

### 커서 기반 (Cursor-Based, 확장성 있는 방식)

```
GET /api/v1/users?cursor=eyJpZCI6MTIzfQ&limit=20

# 구현 예시
SELECT * FROM users
WHERE id > :cursor_id
ORDER BY id ASC
LIMIT 21;  -- 다음 페이지 존재 여부 판단을 위해 하나 더 가져옴
```

```json
{
  "data": [...],
  "meta": {
    "has_next": true,
    "next_cursor": "eyJpZCI6MTQzfQ"
  }
}
```

**장점:** 위치와 상관없이 성능이 일관되며, 데이터 추가 시에도 안정적임
**단점:** 임의의 페이지로 직접 이동할 수 없으며, 커서 값이 불투명함(Opaque)

### 용도별 선택 가이드

| 사용 사례 | 페이지네이션 유형 |
|----------|----------------|
| 관리자 대시보드, 작은 데이터셋 (<1만 건) | 오프셋 방식 |
| 무한 스크롤, 피드, 대량의 데이터셋 | 커서 방식 |
| 공용 API | 커서 방식 (기본) + 오프셋 방식 (선택) |
| 검색 결과 | 오프셋 방식 (사용자가 페이지 번호를 기대함) |

## 필터링, 정렬 및 검색

### 필터링 (Filtering)

```
# 단순 일치
GET /api/v1/orders?status=active&customer_id=abc-123

# 비교 연산자 (대괄호 표기법 권장)
GET /api/v1/products?price[gte]=10&price[lte]=100
GET /api/v1/orders?created_at[after]=2025-01-01

# 여러 값 (콤마로 구분)
GET /api/v1/products?category=electronics,clothing

# 중첩 필드 (점 표기법)
GET /api/v1/orders?customer.country=US
```

### 정렬 (Sorting)

```
# 단일 필드 (내림차순은 접두사 - 사용)
GET /api/v1/products?sort=-created_at

# 여러 필드 (콤마로 구분)
GET /api/v1/products?sort=-featured,price,-created_at
```

### 전체 텍스트 검색 (Full-Text Search)

```
# 검색어 파라미터
GET /api/v1/products?q=wireless+headphones

# 특정 필드 검색
GET /api/v1/users?email=alice
```

### 필수 필드만 선택 (Sparse Fieldsets)

```
# 지정된 필드만 반환 (페이로드 크기 절감)
GET /api/v1/users?fields=id,name,email
GET /api/v1/orders?fields=id,total,status&include=customer.name
```

## 인증 및 권한 부여

### 토큰 기반 인증

```
# Authorization 헤더의 Bearer 토큰
GET /api/v1/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# API 키 (서버 간 통신용)
GET /api/v1/data
X-API-Key: sk_live_abc123
```

### 권한 부여 패턴

```typescript
// 리소스 레벨: 소유권 확인
app.get("/api/v1/orders/:id", async (req, res) => {
  const order = await Order.findById(req.params.id);
  if (!order) return res.status(404).json({ error: { code: "not_found" } });
  if (order.userId !== req.user.id) return res.status(403).json({ error: { code: "forbidden" } });
  return res.json({ data: order });
});

// 역할 기반: 권한 확인
app.delete("/api/v1/users/:id", requireRole("admin"), async (req, res) => {
  await User.delete(req.params.id);
  return res.status(204).send();
});
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
    "message": "속도 제한을 초과했습니다. 60초 후에 다시 시도하십시오."
  }
}
```

### 속도 제한 단계

| 등급 | 제한 | 기준 시간 | 사용 사례 |
|------|-------|--------|----------|
| 익명 | 30회/분 | IP당 | 공용 엔드포인트 |
| 인증됨 | 100회/분 | 사용자당 | 표준 API 액세스 |
| 프리미엄 | 1,000회/분 | API 키당 | 유료 API 플랜 |
| 내부 | 10,000회/분 | 서비스당 | 서비스 간 통신 |

## 버전 관리

### URL 경로 버전 관리 (권장 방식)

```
/api/v1/users
/api/v2/users
```

**장점:** 명확함, 라우팅이 쉬움, 캐싱이 용이함
**단점:** 버전 변경 시 URL이 바뀜

### 헤더 기반 버전 관리

```
GET /api/users
Accept: application/vnd.myapp.v2+json
```

**장점:** 깔끔한 URL
**단점:** 테스트하기 어려움, 누락되기 쉬움

### 버전 관리 전략

```
1. /api/v1/로 시작하십시오 — 필요하기 전까지 무리하게 버전을 나누지 마십시오.
2. 최대 2개의 활성 버전(현재 버전 + 이전 버전)을 유지하십시오.
3. 서비스 종료(Deprecation) 타임라인:
   - 종료 공지 (공용 API의 경우 6개월 전 공지 권장)
   - Sunset 헤더 추가: Sunset: Sat, 01 Jan 2026 00:00:00 GMT
   - 일몰(Sunset) 날짜 이후 410 Gone 반환
4. 하위 호환성을 깨지 않는 변경은 새로운 버전이 필요 없습니다:
   - 응답에 새로운 필드 추가
   - 새로운 선택적 쿼리 파라미터 추가
   - 새로운 엔드포인트 추가
5. 하위 호환성을 깨뜨리는 변경은 새로운 버전이 필요합니다:
   - 필드 삭제 또는 이름 변경
   - 필드 타입 변경
   - URL 구조 변경
   - 인증 방식 변경
```

## 구현 패턴

### TypeScript (Next.js API 라우트)

```typescript
import { z } from "zod";
import { NextRequest, NextResponse } from "next/server";

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});

export async function POST(req: NextRequest) {
  const body = await req.json();
  const parsed = createUserSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json({
      error: {
        code: "validation_error",
        message: "요청 유효성 검사 실패",
        details: parsed.error.issues.map(i => ({
          field: i.path.join("."),
          message: i.message,
          code: i.code,
        })),
      },
    }, { status: 422 });
  }

  const user = await createUser(parsed.data);

  return NextResponse.json(
    { data: user },
    {
      status: 201,
      headers: { Location: `/api/v1/users/${user.id}` },
    },
  );
}
```

### Python (Django REST Framework)

```python
from rest_framework import serializers, viewsets, status
from rest_framework.response import Response

class CreateUserSerializer(serializers.Serializer):
    email = serializers.EmailField()
    name = serializers.CharField(max_length=100)

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ["id", "email", "name", "created_at"]

class UserViewSet(viewsets.ModelViewSet):
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]

    def get_serializer_class(self):
        if self.action == "create":
            return CreateUserSerializer
        return UserSerializer

    def create(self, request):
        serializer = CreateUserSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = UserService.create(**serializer.validated_data)
        return Response(
            {"data": UserSerializer(user).data},
            status=status.HTTP_201_CREATED,
            headers={"Location": f"/api/v1/users/{user.id}"},
        )
```

### Go (net/http)

```go
func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeError(w, http.StatusBadRequest, "invalid_json", "잘못된 요청 본문입니다")
        return
    }

    if err := req.Validate(); err != nil {
        writeError(w, http.StatusUnprocessableEntity, "validation_error", err.Error())
        return
    }

    user, err := h.service.Create(r.Context(), req)
    if err != nil {
        switch {
        case errors.Is(err, domain.ErrEmailTaken):
            writeError(w, http.StatusConflict, "email_taken", "이미 등록된 이메일입니다")
        default:
            writeError(w, http.StatusInternalServerError, "internal_error", "내부 서버 오류")
        }
        return
    }

    w.Header().Set("Location", fmt.Sprintf("/api/v1/users/%s", user.ID))
    writeJSON(w, http.StatusCreated, map[string]any{"data": user})
}
```

## API 설계 체크리스트

새로운 엔드포인트를 배포하기 전 확인 사항:

- [ ] 리소스 URL이 명명 규칙을 따르는가 (복수형, 케밥 케이스, 동사 지양)
- [ ] 올바른 HTTP 메서드를 사용했는가 (조회는 GET, 생성은 POST 등)
- [ ] 적절한 상태 코드를 반환하는가 (모든 응답에 200 사용 금지)
- [ ] 스키마(Zod, Pydantic 등)를 사용하여 입력을 검증했는가
- [ ] 에러 응답이 코드와 메시지를 포함한 표준 형식을 따르는가
- [ ] 목록 엔드포인트에 페이지네이션(커서 또는 오프셋)이 구현되었는가
- [ ] 인증이 필요한가 (또는 명시적으로 공개 엔드포인트로 설정되었는가)
- [ ] 권한 확인이 이루어지는가 (사용자가 자신의 리소스에만 접근 가능한가)
- [ ] 속도 제한이 구성되었는가
- [ ] 응답에 내부 정보(스택 트레이스, SQL 에러 등)가 노출되지 않는가
- [ ] 기존 엔드포인트와 명명 규칙이 일치하는가 (camelCase vs snake_case)
- [ ] 문서(OpenAPI/Swagger 등)가 업데이트되었는가
