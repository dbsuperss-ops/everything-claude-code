---
name: api-design
description: 자원 명명, 상태 코드, 페이지네이션, 필터링, 에러 응답, 버전 관리 및 운영 API의 속도 제한을 포함한 REST API 디자인 패턴입니다.
origin: ECC
---

# API 디자인 패턴

일관성 있고 개발자 친화적인 REST API 설계를 위한 규약과 베스트 프랙티스입니다.

## 적용 시점

* 새로운 API 엔드포인트를 설계할 때
* 기존 API 규약을 검토할 때
* 페이지네이션, 필터링 또는 정렬 기능을 추가할 때
* API의 에러 처리 로직을 구현할 때
* API 버전 관리 전략을 계획할 때
* 외부 공개용 또는 파트너용 API를 구축할 때

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

# 관계를 나타내는 서브 리소스
GET    /api/v1/users/:id/orders
POST   /api/v1/users/:id/orders

# CRUD와 매칭되지 않는 작업 (동사는 가급적 제한적으로 사용)
POST   /api/v1/orders/:id/cancel
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
```

### 명명 규칙

```
# 좋은 예
/api/v1/team-members          # 여러 단어로 된 리소스에는 케밥 케이스 사용
/api/v1/orders?status=active  # 필터링에는 쿼리 파라미터 사용
/api/v1/users/123/orders      # 소유 관계를 나타내는 중첩 리소스

# 나쁜 예
/api/v1/getUsers              # URL에 동사 포함
/api/v1/user                  # 단수형 사용 (복수형 권장)
/api/v1/team_members          # URL에 스네이크 케이스(snake_case) 사용
/api/v1/users/123/getOrders   # 중첩 리소스에 동사 포함
```

## HTTP 메서드 및 상태 코드

### 메서드 의미론

| 메서드 | 멱등성 (Idempotent) | 안전성 (Safe) | 용도 |
|--------|-----------|------|---------|
| GET | 예 | 예 | 리소스 조회 |
| POST | 아니요 | 아니요 | 리소스 생성, 특정 작업 트리거 |
| PUT | 예 | 아니요 | 리소스 전체 교체 |
| PATCH | 아니요* | 아니요 | 리소스 일부 업데이트 |
| DELETE | 예 | 아니요 | 리소스 삭제 |

*적절한 구현을 통해 PATCH도 멱등성을 가질 수 있습니다.

### 상태 코드 참조

```
# 성공 (Success)
200 OK                    — GET, PUT, PATCH (응답 바디 포함 시)
201 Created               — POST (Location 헤더 포함 권장)
204 No Content            — DELETE, PUT (응답 바디 없을 시)

# 클라이언트 에러 (Client Errors)
400 Bad Request           — 검증 실패, 잘못된 형식의 JSON
401 Unauthorized          — 인증 정보 누락 또는 유효하지 않음
403 Forbidden             — 인증은 되었으나 해당 권한 없음
404 Not Found             — 리소스가 존재하지 않음
409 Conflict              — 중복 항목 발견, 상태 충돌
422 Unprocessable Entity  — 의미론적 오류 (JSON 형식은 맞으나 데이터가 부적절함)
429 Too Many Requests     — 속도 제한(Rate limit) 초과

# 서버 에러 (Server Errors)
500 Internal Server Error — 예기치 않은 오류 (민감한 상세 정보 노출 금지)
502 Bad Gateway           — 상위(Upstream) 서비스 오류
503 Service Unavailable   — 일시적인 과부하, Retry-After 헤더 포함 권장
```

### 자주 발생하는 실수

```
# 나쁜 예: 모든 경우에 200 반환
{ "status": 200, "success": false, "error": "Not found" }

# 좋은 예: HTTP 상태 코드를 의미에 맞게 사용
HTTP/1.1 404 Not Found
{ "error": { "code": "not_found", "message": "사용자를 찾을 수 없습니다" } }

# 나쁜 예: 검증 에러에 500 사용
# 좋은 예: 필드별 상세 정보와 함께 400 또는 422 사용

# 나쁜 예: 리소스 생성 시 200 사용
# 좋은 예: Location 헤더와 함께 201 사용
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
    "message": "요청 검증 실패",
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

### 응답 래퍼 변체

```typescript
// 방법 A: 데이터 래퍼를 포함한 봉투(Envelope) (공개 API에 권장)
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

// 방법 B: 평면적인(Flat) 응답 (간결하며, 내부 API에서 흔히 사용)
// 성공 시: 리소스를 직접 반환
// 실패 시: 에러 객체 반환
// HTTP 상태 코드로 구분
```

## 페이지네이션 (Pagination)

### 오프셋 기반 (Offset-based) - 단순함

```
GET /api/v1/users?page=2&per_page=20

# 구현 예시
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 20 OFFSET 20;
```

**장점:** 구현이 쉽고, "n번째 페이지로 이동" 기능을 지원함.
**단점:** 오프셋이 클수록 느려짐 (예: OFFSET 100000), 데이터 추가 시 결과 불일치 발생 가능.

### 커서 기반 (Cursor-based) - 확장성

```
GET /api/v1/users?cursor=eyJpZCI6MTIzfQ&limit=20

# 구현 예시
SELECT * FROM users
WHERE id > :cursor_id
ORDER BY id ASC
LIMIT 21;  -- 다음 페이지 유무를 판단하기 위해 1개 더 가져옴
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

**장점:** 위치에 관계없이 일정한 성능 유지, 데이터 추가 시에도 결과가 안정적임.
**단점:** 임의의 페이지로 바로 이동할 수 없음, 커서가 불투명함(Opaque).

### 어떤 것을 선택해야 할까요?

| 사용 사례 | 페이지네이션 유형 |
|----------|----------------|
| 관리자 대시보드, 작은 데이터셋 (<10K) | 오프셋 |
| 무한 스크롤, 피드형 서비스, 대규모 데이터셋 | 커서 |
| 공개용 API | 커서(기본) + 오프셋(옵션) |
| 검색 결과 | 오프셋 (페이지 번호에 대한 사용자 기대) |

## 필터링, 정렬 및 검색

### 필터링

```
# 단순 일치
GET /api/v1/orders?status=active&customer_id=abc-123

# 비교 연산자 (대괄호 표기법 사용)
GET /api/v1/products?price[gte]=10&price[lte]=100
GET /api/v1/orders?created_at[after]=2025-01-01

# 여러 값 (쉼표로 구분)
GET /api/v1/products?category=electronics,clothing

# 중첩 필드 (점 표기법 사용)
GET /api/v1/orders?customer.country=US
```

### 정렬

```
# 단일 필드 (내림차순은 - 접두사 사용)
GET /api/v1/products?sort=-created_at

# 여러 필드 (쉼표로 구분)
GET /api/v1/products?sort=-featured,price,-created_at
```

### 전체 텍스트 검색

```
# 검색 쿼리 파라미터
GET /api/v1/products?q=wireless+headphones

# 특정 필드 검색
GET /api/v1/users?email=alice
```

### 희소 필드 세트 (Sparse Fieldsets)

```
# 필요한 필드만 지정하여 응답 크기 최적화
GET /api/v1/users?fields=id,name,email
GET /api/v1/orders?fields=id,total,status&include=customer.name
```

## 인증 및 권한 부여

### 토큰 기반 인증

```
# Authorization 헤더에 Bearer 토큰 포함
GET /api/v1/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# API 키 (서버 간 통신 시 사용)
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

# 한도 초과 시
HTTP/1.1 429 Too Many Requests
Retry-After: 60
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "속도 제한을 초과했습니다. 60초 후에 다시 시도하세요."
  }
}
```

### 속도 제한 계층

| 계층 | 한도 | 시간 윈도우 | 사용 사례 |
|------|-------|--------|----------|
| 익명 사용자 | 30/분 | IP당 | 공개 엔드포인트 |
| 인증된 사용자 | 100/분 | 사용자당 | 표준 API 접근 |
| 프리미엄 사용자 | 1000/분 | API 키당 | 유료 API 플랜 |
| 내부 서비스 | 10000/분 | 서비스당 | 서비스 간 통신 |

## 버전 관리

### URL 경로 버전 관리 (권장)

```
/api/v1/users
/api/v2/users
```

**장점:** 명확하고 라우팅이 쉬우며 캐싱이 가능함.
**단점:** 버전 변경 시 URL이 바뀜.

### 헤더 버전 관리

```
GET /api/users
Accept: application/vnd.myapp.v2+json
```

**장점:** 깔끔한 URL 유지 가능.
**단점:** 테스트가 어렵고 누락하기 쉬움.

### 버전 관리 전략

```
1. /api/v1/으로 시작하십시오. 꼭 필요한 상황이 되기 전까지는 버전을 올리지 마세요.
2. 최대 2개의 버전(현재 버전 + 직전 버전)만 활성 상태로 유지하십시오.
3. 서비스 종료(Deprecation) 타임라인:
   - 종료 예고 (공개 API의 경우 최소 6개월 전 공지)
   - Sunset 헤더 추가: Sunset: Sat, 01 Jan 2026 00:00:00 GMT
   - 일몰 기한 이후에는 410 Gone 반환
4. 하위 호환성을 깨지 않는 변경은 새로운 버전이 필요하지 않습니다:
   - 응답에 새로운 필드 추가
   - 새로운 선택적 쿼리 파라미터 추가
   - 새로운 엔드포인트 추가
5. 하위 호환성을 깨는 변경은 새로운 버전이 필요합니다:
   - 기존 필드 삭제 또는 이름 변경
   - 필드 타입 변경
   - URL 구조 변경
   - 인증 방식 변경
```

## 구현 패턴

### TypeScript (Next.js API Routes 예시)

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
        message: "요청 검증 실패",
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

### Python (Django REST Framework 예시)

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

### Go (net/http 예시)

```go
func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeError(w, http.StatusBadRequest, "invalid_json", "잘못된 요청 바디")
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

## API 디자인 체크리스트

새로운 엔드포인트를 공개하기 전에 다음 항목을 확인하십시오:

* [ ] 리소스 URL이 명명 규칙을 따르는가 (복수형, 하이픈 사용, 동사 지양)
* [ ] 올바른 HTTP 메서드를 사용했는가 (조회는 GET, 생성은 POST 등)
* [ ] 상황에 맞는 적절한 상태 코드를 반환하는가 (모든 경우에 200 반환 금지)
* [ ] 스키마(Zod, Pydantic 등)를 사용하여 입력값을 검증했는가
* [ ] 에러 응답이 코드와 메시지를 포함한 표준 형식을 따르는가
* [ ] 목록 엔드포인드에 페이지네이션(커서 또는 오프셋)이 구현되었는가
* [ ] 인증이 필요한가 (또는 공개용으로 명확히 표시되었는가)
* [ ] 권한 부여를 확인했는가 (사용자가 자신의 리소스에만 접근 가능한지)
* [ ] 속도 제한이 구성되었는가
* [ ] 응답에 내부 구현 세부 정보(Stack trace, SQL 에러 등)가 노출되지 않는가
* [ ] 기존 엔드포인트와 명명 방식(camelCase vs snake_case)이 일관적인가
* [ ] 문서화가 완료되었는가 (OpenAPI/Swagger 명세 업데이트)
