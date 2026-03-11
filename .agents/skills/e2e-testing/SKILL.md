---
name: e2e-testing
description: Playwright E2E 테스트 패턴, 페이지 오브젝트 모델(POM), 설정, CI/CD 통합, 산출물 관리 및 불안정한 테스트(Flaky test) 대응 전략을 담고 있습니다.
origin: ECC
---

# E2E 테스트 패턴

안정적이고 빠르며 유지보수가 용이한 E2E 테스트 슈트를 구축하기 위한 포괄적인 Playwright 패턴입니다.

## 테스트 파일 구성

```
tests/
├── e2e/
│   ├── auth/
│   │   ├── login.spec.ts
│   │   ├── logout.spec.ts
│   │   └── register.spec.ts
│   ├── features/
│   │   ├── browse.spec.ts
│   │   ├── search.spec.ts
│   │   └── create.spec.ts
│   └── api/
│       └── endpoints.spec.ts
├── fixtures/
│   ├── auth.ts
│   └── data.ts
└── playwright.config.ts
```

## 페이지 오브젝트 모델 (Page Object Model, POM)

```typescript
import { Page, Locator } from '@playwright/test'

export class ItemsPage {
  readonly page: Page
  readonly searchInput: Locator
  readonly itemCards: Locator
  readonly createButton: Locator

  constructor(page: Page) {
    this.page = page
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
    await this.page.waitForResponse(resp => resp.url().includes('/api/search'))
    await this.page.waitForLoadState('networkidle')
  }

  async getItemCount() {
    return await this.itemCards.count()
  }
}
```

## 테스트 구조

```typescript
import { test, expect } from '@playwright/test'
import { ItemsPage } from '../../pages/ItemsPage'

test.describe('아이템 검색', () => {
  let itemsPage: ItemsPage

  test.beforeEach(async ({ page }) => {
    itemsPage = new ItemsPage(page)
    await itemsPage.goto()
  })

  test('키워드로 검색 가능해야 함', async ({ page }) => {
    await itemsPage.search('test')

    const count = await itemsPage.getItemCount()
    expect(count).toBeGreaterThan(0)

    await expect(itemsPage.itemCards.first()).toContainText(/test/i)
    await page.screenshot({ path: 'artifacts/search-results.png' })
  })

  test('검색 결과가 없을 때를 처리해야 함', async ({ page }) => {
    await itemsPage.search('xyznonexistent123')

    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    expect(await itemsPage.getItemCount()).toBe(0)
  })
})
```

## Playwright 설정

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['junit', { outputFile: 'playwright-results.xml' }],
    ['json', { outputFile: 'playwright-results.json' }]
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10000,
    navigationTimeout: 30000,
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
})
```

## 불안정한 테스트(Flaky Test) 대응 패턴

### 격리 및 보류 (Quarantine)

```typescript
test('flaky: 복잡한 검색', async ({ page }) => {
  test.fixme(true, '불안정함 - 이슈 #123')
  // 테스트 코드...
})

test('조건부 스킵', async ({ page }) => {
  test.skip(process.env.CI, 'CI에서 불안정함 - 이슈 #123')
  // 테스트 코드...
})
```

### 불안정성 확인

```bash
npx playwright test tests/search.spec.ts --repeat-each=10
npx playwright test tests/search.spec.ts --retries=3
```

### 흔한 원인 및 해결책

**경합 조건 (Race conditions):**
```typescript
// 나쁜 예: 요소가 준비되었다고 가정함
await page.click('[data-testid="button"]')

// 좋은 예: 자동 대기(Auto-wait) 로케이터 사용
await page.locator('[data-testid="button"]').click()
```

**네트워크 타이밍:**
```typescript
// 나쁜 예: 임의의 대기 시간 설정
await page.waitForTimeout(5000)

// 좋은 예: 특정 조건이 충족될 때까지 대기
await page.waitForResponse(resp => resp.url().includes('/api/data'))
```

**애니메이션 타이밍:**
```typescript
// 나쁜 예: 애니메이션 도중에 클릭 시도
await page.click('[data-testid="menu-item"]')

// 좋은 예: 상태가 안정될 때까지 대기
await page.locator('[data-testid="menu-item"]').waitFor({ state: 'visible' })
await page.waitForLoadState('networkidle')
await page.locator('[data-testid="menu-item"]').click()
```

## 산출물(Artifact) 관리

### 스크린샷

```typescript
await page.screenshot({ path: 'artifacts/after-login.png' })
await page.screenshot({ path: 'artifacts/full-page.png', fullPage: true })
await page.locator('[data-testid="chart"]').screenshot({ path: 'artifacts/chart.png' })
```

### 트레이스 (Traces)

```typescript
await browser.startTracing(page, {
  path: 'artifacts/trace.json',
  screenshots: true,
  snapshots: true,
})
// ... 테스트 동작 ...
await browser.stopTracing()
```

### 비디오

```typescript
// playwright.config.ts 파일 내 설정
use: {
  video: 'retain-on-failure',
  videosPath: 'artifacts/videos/'
}
```

## CI/CD 통합

```yaml
# .github/workflows/e2e.yml
name: E2E Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
        env:
          BASE_URL: ${{ vars.STAGING_URL }}
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

## 테스트 보고서 템플릿

```markdown
# E2E 테스트 보고서

**날짜:** YYYY-MM-DD HH:MM
**소요 시간:** X분 Y초
**상태:** 통과(PASSING) / 실패(FAILING)

## 요약
- 전체: X | 통과: Y (Z%) | 실패: A | 불안정(Flaky): B | 스킵: C

## 실패한 테스트
### 테스트 이름
**파일:** `tests/e2e/feature.spec.ts:45`
**에러:** 요소가 보일 것으로 예상했으나 보이지 않음
**스크린샷:** artifacts/failed.png
**권장 수정 사항:** [설명]

## 산출물
- HTML 보고서: playwright-report/index.html
- 스크린샷: artifacts/*.png
- 비디오: artifacts/videos/*.webm
- 트레이스: artifacts/*.zip
```

## 지갑 / Web3 테스트

```typescript
test('지갑 연결 테스트', async ({ page, context }) => {
  // 지갑 공급자(Wallet provider) 모킹
  await context.addInitScript(() => {
    window.ethereum = {
      isMetaMask: true,
      request: async ({ method }) => {
        if (method === 'eth_requestAccounts')
          return ['0x1234567890123456789012345678901234567890']
        if (method === 'eth_chainId') return '0x1'
      }
    }
  })

  await page.goto('/')
  await page.locator('[data-testid="connect-wallet"]').click()
  await expect(page.locator('[data-testid="wallet-address"]')).toContainText('0x1234')
})
```

## 금융 / 핵심 서비스 흐름 테스트

```typescript
test('거래 실행 테스트', async ({ page }) => {
  // 실제 비용이 발생하는 운영 환경에서는 스킵
  test.skip(process.env.NODE_ENV === 'production', '운영 환경에서는 건너뜀')

  await page.goto('/markets/test-market')
  await page.locator('[data-testid="position-yes"]').click()
  await page.locator('[data-testid="trade-amount"]').fill('1.0')

  // 미리보기 확인
  const preview = page.locator('[data-testid="trade-preview"]')
  await expect(preview).toContainText('1.0')

  // 승인 및 블록체인 처리 대기
  await page.locator('[data-testid="confirm-trade"]').click()
  await page.waitForResponse(
    resp => resp.url().includes('/api/trade') && resp.status() === 200,
    { timeout: 30000 }
  )

  await expect(page.locator('[data-testid="trade-success"]')).toBeVisible()
})
```
