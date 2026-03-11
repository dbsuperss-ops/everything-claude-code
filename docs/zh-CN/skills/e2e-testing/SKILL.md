---
name: e2e-testing
description: Playwright를 사용한 E2E 테스트 패턴, 페이지 객체 모델(POM), 구성 파일 설정, CI/CD 통합, 테스트 결과물 관리 및 불안정한 테스트(Flaky tests) 대응 전략입니다.
origin: ECC
---

# E2E 테스트 패턴

안정적이고 빠르며 유지보수가 용이한 E2E 테스트 스위트 구축을 위한 Playwright 핵심 패턴 가이드입니다.

## 테스트 파일 구조

```
tests/
├── e2e/                     # E2E 테스트 폴더
│   ├── auth/                # 도메인/기능별 분류
│   │   ├── login.spec.ts
│   │   ├── logout.spec.ts
│   │   └── register.spec.ts
│   ├── features/
│   │   ├── browse.spec.ts
│   │   ├── search.spec.ts
│   │   └── create.spec.ts
│   └── api/                 # API 테스트 (선택 사항)
│       └── endpoints.spec.ts
├── fixtures/                # 공통 테스트 데이터 및 설정
│   ├── auth.ts
│   └── data.ts
└── playwright.config.ts     # Playwright 설정 파일
```

## 페이지 객체 모델 (Page Object Model - POM)

```typescript
import { Page, Locator } from '@playwright/test'

export class ItemsPage {
  readonly page: Page
  readonly searchInput: Locator
  readonly itemCards: Locator
  readonly createButton: Locator

  constructor(page: Page) {
    this.page = page
    // data-testid 속성을 사용하여 견고한 로케이터 정의 권장
    this.searchInput = page.locator('[data-testid="search-input"]')
    this.itemCards = page.locator('[data-testid="item-card"]')
    this.createButton = page.locator('[data-testid="create-btn"]')
  }

  async goto() {
    await this.page.goto('/items')
    await this.page.waitForLoadState('networkidle')
  }

  async search(query: string) {
    await this.searchInput.fill(query)
    // 데이터 로딩 완료 대기 필수
    await this.page.waitForResponse(resp => resp.url().includes('/api/search'))
    await this.page.waitForLoadState('networkidle')
  }

  async getItemCount() {
    return await this.itemCards.count()
  }
}
```

## 테스트 구조 예시

```typescript
import { test, expect } from '@playwright/test'
import { ItemsPage } from '../../pages/ItemsPage'

test.describe('상품 검색 기능', () => {
  let itemsPage: ItemsPage

  test.beforeEach(async ({ page }) => {
    itemsPage = new ItemsPage(page)
    await itemsPage.goto()
  })

  test('키워드로 상품이 검색되어야 함', async ({ page }) => {
    await itemsPage.search('test')

    const count = await itemsPage.getItemCount()
    expect(count).toBeGreaterThan(0) // 결과가 1개 이상 존재해야 함

    await expect(itemsPage.itemCards.first()).toContainText(/test/i)
    await page.screenshot({ path: 'artifacts/search-results.png' })
  })

  test('검색 결과가 없을 때 안내 메시지가 표시됨', async ({ page }) => {
    await itemsPage.search('존재하지않는상품123')

    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    expect(await itemsPage.getItemCount()).toBe(0)
  })
})
```

## Playwright 설정 (playwright.config.ts)

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,             // 모든 테스트 병렬 실행
  forbidOnly: !!process.env.CI,    // CI 환경에서 .only 금지
  retries: process.env.CI ? 2 : 0, // 실패 시 재시도 횟수
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['junit', { outputFile: 'playwright-results.xml' }]
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',       // 실패 시에만 트레이스 기록
    screenshot: 'only-on-failure', // 실패 시에만 스크린샷 저장
    video: 'retain-on-failure',    // 실패 시 동영상 보관
    actionTimeout: 10000,          // 개별 액션 타임아웃
    navigationTimeout: 30000,      // 페이지 이동 타임아웃
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
  webServer: {                     // 테스트 전 로컬 서버 자동 실행
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
})
```

## 불안정한 테스트(Flaky Test) 대응 패턴

### 격리 및 추적

```typescript
test('이슈 #123으로 인해 일시 중단된 테스트', async ({ page }) => {
  test.fixme(true, 'Flaky - 이슈 #123 해결 전까지 수정 필요')
  // 테스트 코드...
})

test('CI 환경에서만 건너뛰는 테스트', async ({ page }) => {
  test.skip(process.env.CI, 'CI 리소스 부족으로 인한 지연 이슈 - #123')
  // 테스트 코드...
})
```

### 불안정성 식별 명령어
```bash
# 특정 테스트를 10번 반복 실행하여 재현 여부 확인
npx playwright test tests/search.spec.ts --repeat-each=10
```

### 흔한 원인과 해결책

**레이스 컨디션 (Race Conditions):**
* 나쁨: 요소가 나타났다고 가정하고 즉시 클릭 (`await page.click(...)`)
* 좋음: 로케이터가 요소를 찾을 때까지 자동 대기 (`await page.locator(...).click()`)

**네트워크 타이밍:**
* 나쁨: `await page.waitForTimeout(5000)` 처럼 임의의 시간 대기 (금지)
* 좋음: 특정 네트워크 응답이 올 때까지 대기 (`await page.waitForResponse(...)`)

## 결과물(Artifacts) 관리

### 스크린샷 및 영상
* 실패 시 스크린샷: `screenshot: 'only-on-failure'`
* 전 페이지 스크린샷: `await page.screenshot({ fullPage: true })`
* 실패 시 동영상: `video: 'retain-on-failure'`

### 트레이스(Trace) 분석
Playwright Trace Viewer를 사용하여 실패 시점의 DOM 상태, 네트워크, 콘솔 로그를 타임라인별로 상세히 분석할 수 있습니다.

## CI/CD 통합 (GitHub Actions)

```yaml
# .github/workflows/e2e.yml
name: E2E 테스트
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx playwright install --with-deps # 브라우저 및 의존성 설치
      - run: npx playwright test
      - uses: actions/upload-artifact@v4 # 결과물 업로드
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

## 테스트 리포트 템플릿 예시

```markdown
# E2E 테스트 결과 보고서

**일시:** 2024-03-10 14:00
**결과:** ❌ 실패 (일부 테스트 실패)

## 요약
- 전체: 50 | 통과: 48 (96%) | 실패: 1 | 불안정: 1 | 건너뜀: 0

## 실패 상세
### 상품 구매 프로세스
- **파일:** `tests/e2e/feature.spec.ts:45`
- **에러:** '결제 완료' 버튼이 나타나지 않음 (타임아웃)
- **스크린샷:** `artifacts/failed.png`
- **원인 분석:** 결제 대행사(PG) 샌드박스 서버 응답 지연으로 판단됨
```

## 특수 케이스 테스트

### Web3 / 지갑 연결 테스트 (Mocking)
```typescript
test('지갑 연결 테스트', async ({ page, context }) => {
  // 브라우저에 가짜 이더리움 객체 주입
  await context.addInitScript(() => {
    window.ethereum = {
      isMetaMask: true,
      request: async ({ method }) => {
        if (method === 'eth_requestAccounts') return ['0x보유주소...']
        if (method === 'eth_chainId') return '0x1'
      }
    }
  })
  // 이후 연결 버튼 클릭 및 주소 표시 확인 로직 수행
})
```

### 금융/결제 중요 로직 테스트
* 운영 환경에서는 실제 결제가 일어날 수 있으므로 반드시 `test.skip(process.env.NODE_ENV === 'production')` 처리를 하십시오.
* 블록체인이나 외부 API 연동 시 타임아웃을 넉넉히 설정하십시오 (`{ timeout: 30000 }`).
