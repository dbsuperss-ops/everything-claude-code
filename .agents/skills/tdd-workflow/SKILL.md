---
name: tdd-workflow
description: 새로운 기능을 작성하거나, 버그를 수정하거나, 코드를 리팩토링할 때 이 스킬을 사용하십시오. 유닛 테스트, 통합 테스트 및 E2E 테스트를 포함하여 80% 이상의 테스트 커버리지를 보장하는 테스트 주도 개발(TDD)을 강제합니다.
origin: ECC
---

# 테스트 주도 개발(TDD) 워크플로우

이 스킬은 포괄적인 테스트 커버리지를 바탕으로 모든 코드 개발이 TDD 원칙을 따르도록 보장합니다.

## 활성화 시기

- 새로운 기능 또는 기능을 작성할 때
- 버그 또는 이슈를 수정할 때
- 기존 코드를 리팩토링할 때
- API 엔드포인트를 추가할 때
- 새로운 컴포넌트를 생성할 때

## 핵심 원칙

### 1. 코드보다 테스트 먼저 (Tests BEFORE Code)
항상 테스트를 먼저 작성한 다음, 해당 테스트를 통과시키기 위한 코드를 구현하십시오.

### 2. 커버리지 요구 사항
- 최소 80% 커버리지 확보 (유닛 + 통합 + E2E 합계)
- 모든 이례적인 상황(Edge cases) 포함
- 에러 시나리오 테스트 포함
- 경계값 조건(Boundary conditions) 검증

### 3. 테스트 유형

#### 유닛 테스트 (Unit Tests)
- 개별 함수 및 유틸리티
- 컴포넌트 로직
- 순수 함수 (Pure functions)
- 헬퍼 및 유틸리티

#### 통합 테스트 (Integration Tests)
- API 엔드포인트
- 데이터베이스 작업
- 서비스 간 상호작용
- 외부 API 호출

#### E2E 테스트 (Playwright 사용)
- 핵심 사용자 흐름 (Critical user flows)
- 전체 워크플로우 완료 여부
- 브라우저 자동화
- UI 상호작용

## TDD 워크플로우 단계

### 1단계: 사용자 여정(User Journeys) 작성
```
[역할]로서, 나는 [이득]을 위해 [행동]을 하고 싶다.

예시:
사용자로서, 나는 정확한 키워드가 없어도 관련 마켓을 찾을 수 있도록 
의미 기반으로 마켓을 검색하고 싶다.
```

### 2단계: 테스트 케이스 생성
각 사용자 여정에 대해 포괄적인 테스트 케이스를 생성합니다:

```typescript
describe('의미 기반 검색(Semantic Search)', () => {
  it('쿼리에 대해 관련성 있는 마켓을 반환함', async () => {
    // 테스트 구현
  })

  it('빈 쿼리를 우아하게 처리함', async () => {
    // 에지 케이스 테스트
  })

  it('Redis를 사용할 수 없을 경우 부분 문자열 검색으로 대체함', async () => {
    // 대체 동작(Fallback) 테스트
  })

  it('유사도 점수에 따라 결과를 정렬함', async () => {
    // 정렬 로직 테스트
  })
})
```

### 3단계: 테스트 실행 (실패해야 함)
```bash
npm test
# 아직 코드를 구현하지 않았으므로 테스트가 실패해야 합니다.
```

### 4단계: 코드 구현
테스트를 통과하기 위한 최소한의 코드를 작성합니다:

```typescript
// 테스트에 기반한 구현
export async function searchMarkets(query: string) {
  // 여기에 구현 내용 작성
}
```

### 5단계: 테스트 다시 실행
```bash
npm test
# 이제 테스트가 통과해야 합니다.
```

### 6단계: 리팩토링
테스트 통과 상태를 유지하면서 코드 품질을 개선합니다:
- 중복 제거
- 이름 명확화
- 성능 최적화
- 가독성 향상

### 7단계: 커버리지 확인
```bash
npm run test:coverage
# 80% 이상의 커버리지가 달성되었는지 확인합니다.
```

## 테스트 패턴

### 유닛 테스트 패턴 (Jest/Vitest 사용)
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('Button 컴포넌트', () => {
  it('정확한 텍스트와 함께 렌더링됨', () => {
    render(<Button>클릭하세요</Button>)
    expect(screen.getByText('클릭하세요')).toBeInTheDocument()
  })

  it('클릭 시 onClick이 호출됨', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>클릭</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('disabled prop이 true일 때 비활성화됨', () => {
    render(<Button disabled>클릭</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### API 통합 테스트 패턴
```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('성공적으로 마켓을 반환함', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('쿼리 파라미터를 검증함', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('데이터베이스 에러를 우아하게 처리함', async () => {
    // 데이터베이스 실패 모킹(Mocking)
    const request = new NextRequest('http://localhost/api/markets')
    // 에러 처리 테스트
  })
})
```

### E2E 테스트 패턴 (Playwright 사용)
```typescript
import { test, expect } from '@playwright/test'

test('사용자가 마켓을 검색하고 필터링할 수 있음', async ({ page }) => {
  // 마켓 페이지로 이동
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // 페이지 로드 확인
  await expect(page.locator('h1')).toContainText('Markets')

  // 마켓 검색
  await page.fill('input[placeholder="Search markets"]', 'election')

  // 디바운스 및 결과 대기
  await page.waitForTimeout(600)

  // 검색 결과가 표시되는지 확인
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 결과에 검색어가 포함되었는지 확인
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // 상태별 필터링
  await page.click('button:has-text("Active")')

  // 필터링된 결과 확인
  await expect(results).toHaveCount(3)
})

test('사용자가 새로운 마켓을 생성할 수 있음', async ({ page }) => {
  // 먼저 로그인 페이지 또는 대시보드로 이동
  await page.goto('/creator-dashboard')

  // 마켓 생성 폼 작성
  await page.fill('input[name="name"]', '테스트 마켓')
  await page.fill('textarea[name="description"]', '테스트 설명')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // 폼 제출
  await page.click('button[type="submit"]')

  // 성공 메시지 확인
  await expect(page.locator('text=마켓이 성공적으로 생성되었습니다')).toBeVisible()

  // 마켓 페이지로 리다이렉트되었는지 확인
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## 테스트 파일 구성

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # 유닛 테스트
│   │   └── Button.stories.tsx       # 스토리북
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # 통합 테스트
└── e2e/
    ├── markets.spec.ts               # E2E 테스트
    ├── trading.spec.ts
    └── auth.spec.ts
```

## 외부 서비스 모킹 (Mocking)

### Supabase 모킹
```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: [{ id: 1, name: '테스트 마켓' }],
          error: null
        }))
      }))
    }))
  }
}))
```

### Redis 모킹
```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ])),
  checkRedisHealth: jest.fn(() => Promise.resolve({ connected: true }))
}))
```

### OpenAI 모킹
```typescript
jest.mock('@/lib/openai', () => ({
  generateEmbedding: jest.fn(() => Promise.resolve(
    new Array(1536).fill(0.1) // 1536차원 임베딩 모킹
  ))
}))
```

## 테스트 커버리지 확인

### 커버리지 보고서 실행
```bash
npm run test:coverage
```

### 커버리지 임계값(Thresholds) 설정
```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## 지양해야 할 흔한 테스트 실수

### ❌ 잘못된 예: 구현 상세 내용 테스트
```typescript
// 내부 상태를 테스트하지 마십시오
expect(component.state.count).toBe(5)
```

### ✅ 올바른 예: 사용자에게 보이는 동작 테스트
```typescript
// 사용자가 보는 내용을 테스트하십시오
expect(screen.getByText('Count: 5')).toBeInTheDocument()
```

### ❌ 잘못된 예: 부서지기 쉬운 선택자(Selectors)
```typescript
// 쉽게 부서질 수 있는 방식
await page.click('.css-class-xyz')
```

### ✅ 올바른 예: 의미론적 선택자
```typescript
// 변경에 강한 방식
await page.click('button:has-text("제출")')
await page.click('[data-testid="submit-button"]')
```

### ❌ 잘못된 예: 테스트 간 격리 부족
```typescript
// 테스트들이 서로 의존함
test('사용자 생성', () => { /* ... */ })
test('동일 사용자 업데이트', () => { /* 이전 테스트의 결과에 의존함 */ })
```

### ✅ 올바른 예: 독립적인 테스트
```typescript
// 각 테스트가 자신의 데이터를 직접 설정함
test('사용자 생성', () => {
  const user = createTestUser()
  // 테스트 로직
})

test('사용자 업데이트', () => {
  const user = createTestUser()
  // 업데이트 로직
})
```

## 지속적 테스트 (Continuous Testing)

### 개발 도중 감시(Watch) 모드
```bash
npm test -- --watch
# 파일 변경 시 테스트가 자동으로 실행됨
```

### 프리커밋(Pre-Commit) 후크
```bash
# 커밋 전마다 실행됨
npm test && npm run lint
```

### CI/CD 통합
```yaml
# GitHub Actions 예시
- name: Run Tests
  run: npm test -- --coverage
- name: Upload Coverage
  uses: codecov/codecov-action@v3
```

## 모범 사례

1. **테스트 먼저 작성하기** - 항상 TDD를 실천하십시오.
2. **테스트당 하나의 단언(Assert) 사용** - 하나의 동작에 집중하십시오.
3. **설명적인 테스트 이름** - 무엇이 테스트되는지 명확히 하십시오.
4. **준비-실행-검증(Arrange-Act-Assert)** - 명확한 테스트 구조를 갖추십시오.
5. **외부 의존성 모킹** - 유닛 테스트를 격리하십시오.
6. **에지 케이스 테스트** - null, undefined, 빈 값, 대용량 데이터 등을 포함하십시오.
7. **에러 경로 테스트** - 정상적인 경로뿐만 아니라 에러 발생 시의 경로도 테스트하십시오.
8. **테스트 속도 유지** - 유닛 테스트는 각각 50ms 이내여야 합니다.
9. **테스트 후 정리** - 부수 효과(Side effects)가 남지 않도록 하십시오.
10. **커버리지 보고서 활용** - 부족한 영역을 식별하십시오.

## 성공 지표

- 80% 이상의 코드 커버리지 달성
- 모든 테스트 통과 (초록불)
- 스킵되거나 비활성화된 테스트 없음
- 테스트 실행 속도 확보 (유닛 테스트 전체 30초 이내)
- E2E 테스트가 핵심 사용자 여정을 포함함
- 상용 배포 전 버그를 테스트에서 잡아냄

---

**기억해 두세요**: 테스트는 선택 사항이 아닙니다. 테스트는 자신감 있는 리팩토링, 신속한 개발 및 운영 신뢰성을 보장하는 안전그물입니다.
