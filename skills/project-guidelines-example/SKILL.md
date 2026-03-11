---
name: project-guidelines-example
description: "실제 프로덕션 애플리케이션을 기반으로 한 예시 프로젝트 전용 스킬 템플릿입니다."
origin: ECC
---

# 프로젝트 가이드라인 스킬 (예시)

이것은 프로젝트 전용 스킬의 예시입니다. 자신의 프로젝트를 위한 템플릿으로 사용하십시오.

실제 프로덕션 애플리케이션인 [Zenith](https://zenith.chat) (AI 기반 고객 발굴 플랫폼)를 기반으로 합니다.

## 사용 시점

이 스킬이 설계된 특정 프로젝트에서 작업할 때 참조하십시오. 프로젝트 스킬은 다음을 포함합니다:
- 아키텍처 개요
- 파일 구조
- 코드 패턴
- 테스트 요구 사항
- 배포 워크플로우

---

## 아키텍처 개요

**기술 스택:**
- **프론트엔드**: Next.js 15 (App Router), TypeScript, React
- **백엔드**: FastAPI (Python), Pydantic 모델
- **데이터베이스**: Supabase (PostgreSQL)
- **AI**: 도구 호출(Tool calling) 및 구조화된 출력을 사용하는 Claude API
- **배포**: Google Cloud Run
- **테스트**: Playwright (E2E), pytest (백엔드), React Testing Library

**서비스:**
```
┌─────────────────────────────────────────────────────────────┐
│                        프론트엔드                             │
│  Next.js 15 + TypeScript + TailwindCSS                     │
│  배포: Vercel / Cloud Run                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         백엔드                               │
│  FastAPI + Python 3.11 + Pydantic                          │
│  배포: Cloud Run                                            │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Supabase │   │  Claude  │   │  Redis   │
        │ 데이터베이스 │   │   API    │   │  캐시     │
        └──────────┘   └──────────┘   └──────────┘
```

---

## 파일 구조

```
project/
├── frontend/
│   └── src/
│       ├── app/              # Next.js 앱 라우터 페이지
│       │   ├── api/          # API 라우트
│       │   ├── (auth)/       # 인증 보호 라우트
│       │   └── workspace/    # 메인 앱 워크스페이스
│       ├── components/       # React 컴포넌트
│       │   ├── ui/           # 기본 UI 컴포넌트
│       │   ├── forms/        # 폼 컴포넌트
│       │   └── layouts/      # 레이아웃 컴포넌트
│       ├── hooks/            # 커스텀 React 훅
│       ├── lib/              # 유틸리티
│       ├── types/            # TypeScript 정의
│       └── config/           # 설정
│
├── backend/
│   ├── routers/              # FastAPI 라우트 핸들러
│   ├── models.py             # Pydantic 모델
│   ├── main.py               # FastAPI 앱 엔트리
│   ├── auth_system.py        # 인증
│   ├── database.py           # 데이터베이스 작업
│   ├── services/             # 비즈니스 로직
│   └── tests/                # pytest 테스트
│
├── deploy/                   # 배포 설정
├── docs/                     # 문서
└── scripts/                  # 유틸리티 스크립트
```

---

## 코드 패턴

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

### 프론트엔드 API 호출 (TypeScript)

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
      return { success: false, error: `HTTP ${response.status}` }
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
        model="claude-sonnet-4-5-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": content}],
        tools=[{
            "name": "provide_analysis",
            "description": "구조화된 분석 제공",
            "input_schema": AnalysisResult.model_json_schema()
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

### 커스텀 훅 (React)

```typescript
import { useState, useCallback } from 'react'

interface UseApiState<T> {
  data: T | null
  loading: boolean
  error: string | null
}

export function useApi<T>(
  fetchFn: () => Promise<ApiResponse<T>>
) {
  const [state, setState] = useState<UseApiState<T>>({
    data: null,
    loading: false,
    error: null,
  })

  const execute = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }))

    const result = await fetchFn()

    if (result.success) {
      setState({ data: result.data!, loading: false, error: null })
    } else {
      setState({ data: null, loading: false, error: result.error! })
    }
  }, [fetchFn])

  return { ...state, execute }
}
```

---

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

**테스트 구조:**
```python
import pytest
from httpx import AsyncClient
from main import app

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

@pytest.mark.asyncio
async def test_health_check(client: AsyncClient):
    response = await client.get("/health")
    assert response.status_code == 200
    assert response.json()["status"] == "healthy"
```

### 프론트엔드 (React Testing Library)

```bash
# 테스트 실행
npm run test

# 커버리지와 함께 실행
npm run test -- --coverage

# E2E 테스트 실행
npm run test:e2e
```

**테스트 구조:**
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { WorkspacePanel } from './WorkspacePanel'

describe('WorkspacePanel', () => {
  it('워크스페이스를 올바르게 렌더링함', () => {
    render(<WorkspacePanel />)
    expect(screen.getByRole('main')).toBeInTheDocument()
  })

  it('세션 생성을 처리함', async () => {
    render(<WorkspacePanel />)
    fireEvent.click(screen.getByText('New Session'))
    expect(await screen.findByText('Session created')).toBeInTheDocument()
  })
})
```

---

## 배포 워크플로우

### 배포 전 체크리스트

- [ ] 로컬에서 모든 테스트 통과
- [ ] `npm run build` 성공 (프론트엔드)
- [ ] `poetry run pytest` 통과 (백엔드)
- [ ] 하드코딩된 비밀 정보 없음
- [ ] 환경 변수 문서화 완료
- [ ] 데이터베이스 마이그레이션 준비 완료

### 배포 명령어

```bash
# 프론트엔드 빌드 및 배포
cd frontend && npm run build
gcloud run deploy frontend --source .

# 백엔드 빌드 및 배포
cd backend
gcloud run deploy backend --source .
```

### 환경 변수

```bash
# 프론트엔드 (.env.local)
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...

# 백엔드 (.env)
DATABASE_URL=postgresql://...
ANTHROPIC_API_KEY=sk-ant-...
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_KEY=eyJ...
```

---

## 핵심 규칙

1. 코드, 주석, 문서에 **이모지 사용 금지**
2. **불변성(Immutability)** — 객체나 배열을 절대 직접 수정하지 마십시오.
3. **TDD** — 구현 전에 테스트를 먼저 작성하십시오.
4. 최소 **80% 커버리지** 유지
5. **작은 파일 여러 개** — 보통 200~400라인, 최대 800라인
6. 프로덕션 코드에 **console.log 금지**
7. try/catch를 이용한 **적절한 에러 처리**
8. Pydantic/Zod를 이용한 **입력값 검증**

---

## 관련 스킬

- `coding-standards.md` - 일반적인 코딩 최선 관행
- `backend-patterns.md` - API 및 데이터베이스 패턴
- `frontend-patterns.md` - React 및 Next.js 패턴
- `tdd-workflow/` - 테스트 주도 개발 방법론
