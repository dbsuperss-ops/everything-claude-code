---
name: project-guidelines-example
description: 실제 프로덕션 애플리케이션을 기반으로 한 프로젝트 전용 스킬 템플릿 예시입니다.
origin: ECC
---

# 프로젝트 가이드라인 스킬 (예시)

이 파일은 프로젝트 전용 스킬의 예시입니다. 여러분의 실제 프로젝트를 위한 템플릿으로 활용하십시오.

본 예시는 AI 기반 고객 발굴 플랫폼인 [Zenith](https://zenith.chat)라는 실제 프로덕션 애플리케이션을 바탕으로 작성되었습니다.

## 적용 시점

이 스킬은 해당 프로젝트를 수행할 때 참고하십시오. 프로젝트 전용 스킬은 다음 내용을 포함합니다:

* 아키텍처 개요
* 파일 구조
* 코드 패턴
* 테스트 요구 사항
* 배포 워크플로우

***

## 아키텍처 개요 (Architecture Overview)

**기술 스택:**

* **프런트엔드**: Next.js 15 (App Router), TypeScript, React
* **백엔드**: FastAPI (Python), Pydantic 모델
* **데이터베이스**: Supabase (PostgreSQL)
* **AI**: Claude API (도구 호출 및 구조화된 출력 지원)
* **배포**: Google Cloud Run
* **테스트**: Playwright (E2E), pytest (백엔드), React Testing Library

**서비스 구성 도표:**

```
┌─────────────────────────────────────────────────────────────┐
│                       프런트엔드 (Frontend)                 │
│  Next.js 15 + TypeScript + TailwindCSS                     │
│  배포: Vercel / Cloud Run                                   │
└─────────────────────────────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                        백엔드 (Backend)                     │
│  FastAPI + Python 3.11 + Pydantic                          │
│  배포: Cloud Run                                            │
└─────────────────────────────────────────────────────────────┘
                               │
               ┌───────────────┼───────────────┐
               ▼               ▼               ▼
         ┌──────────┐   ┌──────────┐   ┌──────────┐
         │ Supabase │   │  Claude  │   │  Redis   │
         │ Database │   │   API    │   │  Cache   │
         └──────────┘   └──────────┘   └──────────┘
```

***

## 파일 구조 (File Structure)

```
project/
├── frontend/
│   └── src/
│       ├── app/              # Next.js 앱 라우터 페이지
│       │   ├── api/          # 내부 API 라우트
│       │   ├── (auth)/       # 인증 보호 라우트
│       │   └── workspace/    # 메인 앱 워크스페이스
│       ├── components/       # React 컴포넌트
│       │   ├── ui/           # 기본 UI 컴포넌트 (Shadcn 등)
│       │   ├── forms/        # 폼 관련 컴포넌트
│       │   └── layouts/      # 레이아웃 컴포넌트
│       ├── hooks/            # 커스텀 React 훅
│       ├── lib/              # 유틸리티 함수
│       ├── types/            # TypeScript 타입 정의
│       └── config/           # 설정 파일
│
├── backend/
│   ├── routers/              # FastAPI 라우트 핸들러
│   ├── models.py             # Pydantic 모델 정의
│   ├── main.py               # FastAPI 앱 엔트리 포인트
│   ├── auth_system.py        # 인증 시스템
│   ├── database.py           # 데이터베이스 작업
│   ├── services/             # 비즈니스 로직 처리
│   └── tests/                # pytest 테스트 코드
│
├── deploy/                   # 배포 설정 파일
├── docs/                     # 프로젝트 문서
└── scripts/                  # 유틸리티 스크립트
```

***

## 코드 패턴 (Code Patterns)

### API 응답 형식 (FastAPI)

```python
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None

    @classmethod
    def ok(cls, data: T) -> "ApiResponse[T]":
        return cls(success=True, data=data)

    @classmethod
    def fail(cls, error: str) -> "ApiResponse[T]":
        return cls(success=False, error=error)
```

### 프런트엔드 API 호출 (TypeScript)

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
}

async function fetchApi<T>(
  endpoint: string,
  options?: RequestInit
): Promise<ApiResponse<T>> {
  try {
    const response = await fetch(`/api${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options?.headers,
      },
    })

    if (!response.ok) {
      return { success: false, error: `HTTP 에러 ${response.status}` }
    }

    return await response.json()
  } catch (error) {
    return { success: false, error: String(error) }
  }
}
```

### Claude AI 통합 (구조화된 출력)

```python
from anthropic import Anthropic
from pydantic import BaseModel

class AnalysisResult(BaseModel):
    summary: str
    key_points: list[str]
    confidence: float

async def analyze_with_claude(content: str) -> AnalysisResult:
    client = Anthropic()

    response = client.messages.create(
        model="claude-3-5-sonnet-20240620",
        max_tokens=1024,
        messages=[{"role": "user", "content": content}],
        tools=[{
            "name": "provide_analysis",
            "description": "구조화된 분석 결과 제공",
            "input_schema": AnalysisResult.model_config
        }],
        tool_choice={"type": "tool", "name": "provide_analysis"}
    )

    # 도구 사용 결과 추출
    tool_use = next(
        block for block in response.content
        if block.type == "tool_use"
    )

    return AnalysisResult(**tool_use.input)
```

***

## 테스트 요구 사항

### 백엔드 (pytest)

```bash
# 모든 테스트 실행
poetry run pytest tests/

# 커버리지와 함께 실행
poetry run pytest tests/ --cov=. --cov-report=html

# 특정 테스트 파일 실행
poetry run pytest tests/test_auth.py -v
```

### 프런트엔드 (React Testing Library)

```bash
# 테스트 실행
npm run test

# 커버리지와 함께 실행
npm run test -- --coverage

# E2E 테스트 실행
npm run test:e2e
```

***

## 배포 워크플로우

### 배포 전 체크리스트
* [ ] 로컬에서 모든 테스트 통과 확인
* [ ] 프런트엔드 `npm run build` 성공 확인
* [ ] 백엔드 `poetry run pytest` 통과 확인
* [ ] 하드코딩된 비밀 정보가 없는지 확인
* [ ] 환경 변수 문서화 완료
* [ ] 데이터베이스 마이그레이션 준비 완료

### 배포 명령어
```bash
# 프런트엔드 빌드 및 배포
cd frontend && npm run build
gcloud run deploy frontend --source .

# 백엔드 빌드 및 배포
cd backend
gcloud run deploy backend --source .
```

***

## 핵심 규칙 (Key Rules)

1. **이모지 사용 금지**: 코드, 주석, 문서 어디에도 이모지를 사용하지 마십시오.
2. **불변성(Immutability)**: 객체나 배열을 직접 변경하지 마십시오.
3. **테스트 주도 개발(TDD)**: 구현 전에 반드시 테스트를 먼저 작성하십시오.
4. **최소 80% 커버리지**: 테스트 커버리지를 80% 이상 유지하십시오.
5. **작은 파일 유지**: 일반적인 파일 크기는 200~400줄이며, 최대 800줄을 넘지 않아야 합니다.
6. **console.log 금지**: 프로덕션 코드에 로그 출력문을 남기지 마십시오.
7. **적절한 에러 처리**: 모든 예외 상황에 대해 try/catch를 사용하십시오.
8. **입력값 검증**: Pydantic(백엔드) 또는 Zod(프런트엔드)를 사용해 모든 입력을 검증하십시오.

***

## 관련 스킬

* `coding-standards.md` - 범용 코딩 베스트 프랙티스
* `backend-patterns.md` - API 및 데이터베이스 패턴
* `frontend-patterns.md` - React 및 Next.js 패턴
* `tdd-workflow/` - 테스트 주도 개발 방법론
