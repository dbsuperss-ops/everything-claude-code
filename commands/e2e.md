---
description: Playwright를 사용하여 엔드투엔드(E2E) 테스트를 생성하고 실행합니다. 테스트 여정 생성, 테스트 실행, 스크린샷/비디오/트레이스 캡처 및 아티팩트 업로드를 수행합니다.
---

# E2E 명령어

이 명령어는 **e2e-runner** 에이전트를 호출하여 Playwright를 사용한 엔드투엔드 테스트를 생성, 유지 관리 및 실행합니다.

## 주요 기능

1. **테스트 여정 생성** - 사용자 흐름에 대한 Playwright 테스트 생성
2. **E2E 테스트 실행** - 다양한 브라우저에서 테스트 실행
3. **아티팩트 캡처** - 실패 시 스크린샷, 비디오, 트레이스(Trace) 저장
4. **결과 업로드** - HTML 보고서 및 JUnit XML 생성
5. **불안정한 테스트(Flaky Tests) 식별** - 불안정한 테스트 격리 및 관리

## 사용 시점

다음과 같은 경우 `/e2e`를 사용하십시오:
- 핵심 사용자 여정(로그인, 거래, 결제 등) 테스트 시
- 여러 단계의 흐름이 엔드투엔드로 작동하는지 확인 시
- UI 상호작용 및 내비게이션 테스트 시
- 프론트엔드와 백엔드 간의 통합 검증 시
- 프로덕션 배포 준비 시

## 작동 원리

e2e-runner 에이전트는 다음을 수행합니다:

1. **사용자 흐름 분석** 및 테스트 시나리오 식별
2. Page Object Model 패턴을 사용하여 **Playwright 테스트 생성**
3. 여러 브라우저(Chrome, Firefox, Safari)에서 **테스트 실행**
4. 스크린샷, 비디오, 트레이스를 통해 **실패 사례 캡처**
5. 결과 및 아티팩트가 포함된 **보고서 생성**
6. **불안정한 테스트 식별** 및 수정 제안

## 사용 예시

```
사용자: /e2e 시장 검색 및 조회 흐름 테스트해줘

에이전트 (e2e-runner):
# E2E 테스트 생성: 시장 검색 및 조회 흐름

## 식별된 테스트 시나리오

**사용자 여정:** 시장 검색 → 결과 조회 → 시장 클릭 → 상세 정보 조회

**테스트 범위:**
1. 시장 페이지로 이동
2. 시맨틱 검색 수행
3. 검색 결과 확인
4. 첫 번째 결과 클릭
5. 시장 상세 페이지 로드 확인
6. 차트 렌더링 확인

## 생성된 테스트 코드

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('시장 검색 및 조회 흐름', () => {
  test('사용자는 시장을 검색하고 상세 정보를 볼 수 있음', async ({ page }) => {
    // 1. 시장 페이지로 이동
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 페이지 로드 확인
    await expect(page).toHaveTitle(/Markets/)
    await expect(page.locator('h1')).toContainText('Markets')

    // 2. 시맨틱 검색 수행
    await marketsPage.searchMarkets('선거')

    // API 응답 대기
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. 검색 결과 확인
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // 검색 결과 스크린샷 캡처
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. 첫 번째 결과 클릭
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. 시장 상세 페이지 로드 확인
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. 차트 렌더링 확인
    await expect(detailsPage.priceChart).toBeVisible()

    // 시장 이름 일치 확인
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // 시장 상세 정보 스크린샷 캡처
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('검색 결과가 없을 때 빈 상태 표시', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 존재하지 않는 시장 검색
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // 빈 상태 확인
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /결과가 없습니다|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('검색어를 지우고 다시 모든 시장을 볼 수 있음', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 초기 시장 개수
    const initialCount = await marketsPage.marketCards.count()

    // 검색 수행
    await marketsPage.searchMarkets('트럼프')
    await page.waitForLoadState('networkidle')

    // 필터링된 결과 확인
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // 검색어 삭제
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // 모든 시장이 다시 표시되는지 확인
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
```

## 테스트 실행

```bash
# 생성된 테스트 실행
npx playwright test tests/e2e/markets/search-and-view.spec.ts

Running 3 tests using 3 workers

  ✓  [chromium] › search-and-view.spec.ts:5:3 › 사용자는 시장을 검색하고 상세 정보를 볼 수 있음 (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › 검색 결과가 없을 때 빈 상태 표시 (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › 검색어를 지우고 다시 모든 시장을 볼 수 있음 (2.9s)

  3 passed (9.1s)

생성된 아티팩트:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## 테스트 보고서

```
╔══════════════════════════════════════════════════════════════╗
║                    E2E 테스트 결과                             ║
╠══════════════════════════════════════════════════════════════╣
║ 상태:       ✅ 모든 테스트 통과                                 ║
║ 총계:       3개 테스트                                        ║
║ 통과:       3 (100%)                                         ║
║ 실패:       0                                                ║
║ 불안정:     0                                                ║
║ 소요 시간:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝

아티팩트:
📸 스크린샷: 2개 파일
📹 비디오: 0개 파일 (실패 시에만 생성)
🔍 트레이스: 0개 파일 (실패 시에만 생성)
📊 HTML 보고서: playwright-report/index.html

보고서 보기: npx playwright show-report
```

✅ CI/CD 통합을 위한 E2E 테스트 스위트 준비 완료!
```

## 테스트 아티팩트

테스트 실행 시 다음과 같은 아티팩트가 캡처됩니다:

**모든 테스트 시:**
- 타임라인과 결과가 포함된 HTML 보고서
- CI 통합을 위한 JUnit XML

**실패 시에만:**
- 실패 시점의 스크린샷
- 테스트 실행 비디오 녹화
- 디버깅을 위한 트레이스 파일 (단계별 리플레이 가능)
- 네트워크 로그
- 콘솔 로그

## 아티팩트 확인 방법

```bash
# 브라우저에서 HTML 보고서 보기
npx playwright show-report

# 특정 트레이스 파일 보기
npx playwright show-trace artifacts/trace-abc123.zip

# 스크린샷은 artifacts/ 디렉토리에 저장됩니다.
open artifacts/search-results.png
```

## 불안정한 테스트(Flaky Test) 감지

테스트가 간헐적으로 실패하는 경우:

```
⚠️  불안정한 테스트 감지됨: tests/e2e/markets/trade.spec.ts

테스트 통과율: 10/7회 (70%)

공통 실패 사유:
"Timeout waiting for element '[data-testid="confirm-btn"]'"

권장 수정 사항:
1. 명시적 대기 추가: await page.waitForSelector('[data-testid="confirm-btn"]')
2. 타임아웃 증가: { timeout: 10000 }
3. 컴포넌트 내 경쟁 상태(Race condition) 확인
4. 애니메이션에 의해 요소가 가려지지 않았는지 확인

격리 권장: 수정될 때까지 test.fixme()로 표시
```

## 브라우저 설정

테스트는 기본적으로 다음 브라우저에서 실행됩니다:
- ✅ Chromium (데스크톱 Chrome)
- ✅ Firefox (데스크톱)
- ✅ WebKit (데스크톱 Safari)
- ✅ Mobile Chrome (선택 사항)

브라우저 조정을 위해 `playwright.config.ts`를 설정하십시오.

## CI/CD 통합

CI 파이프라인에 추가하십시오:

```yaml
# .github/workflows/e2e.yml
- name: Install Playwright
  run: npx playwright install --with-deps

- name: Run E2E tests
  run: npx playwright test

- name: Upload artifacts
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX 전용 핵심 흐름

PMX의 경우 다음 E2E 테스트를 우선시하십시오:

**🔴 치명적 (반드시 통과해야 함):**
1. 사용자가 지갑을 연결할 수 있음
2. 사용자가 시장을 둘러볼 수 있음
3. 사용자가 시장을 검색할 수 있음 (시맨틱 검색)
4. 사용자가 시장 상세 정보를 볼 수 있음
5. 사용자가 거래를 체결할 수 있음 (테스트 자금 사용)
6. 시장이 올바르게 정산됨
7. 사용자가 자금을 출금할 수 있음

**🟡 중요:**
1. 시장 생성 흐름
2. 사용자 프로필 업데이트
3. 실시간 가격 업데이트
4. 차트 렌더링
5. 시장 필터링 및 정렬
6. 모바일 반응형 레이아웃

## 최선 관행 (Best Practices)

**권장 사항 (DO):**
- ✅ 유지보수성을 위해 Page Object Model 사용
- ✅ 선택자(Selector)에 data-testid 속성 사용
- ✅ 임의의 타임아웃이 아닌 API 응답 대기 사용
- ✅ 핵심 사용자 여정을 엔드투엔드로 테스트
- ✅ 메인 브랜치 머지 전 테스트 실행
- ✅ 테스트 실패 시 아티팩트 검토

**지양 사항 (DON'T):**
- ❌ 깨지기 쉬운 선택자 사용 (CSS 클래스는 변경될 수 있음)
- ❌ 구현 세부 사항 테스트
- ❌ 프로덕션 환경에서 테스트 실행
- ❌ 불안정한 테스트 방치
- ❌ 실패 시 아티팩트 검토 생략
- ❌ 모든 엣지 케이스를 E2E로 테스트 (단위 테스트 활용)

## 주의 사항

**PMX 치명적 사항:**
- 실제 자금이 포함된 E2E 테스트는 반드시 테스트넷/스테이징에서만 실행해야 합니다.
- 절대 프로덕션 환경에서 거래 테스트를 실행하지 마십시오.
- 금융 관련 테스트에는 `test.skip(process.env.NODE_ENV === 'production')`을 설정하십시오.
- 소액의 테스트 자금이 든 테스트 지갑만 사용하십시오.

## 다른 명령어와의 통합

- `/plan`을 사용하여 테스트할 핵심 여정 식별
- `/tdd`를 사용하여 단위 테스트 수행 (더 빠르고 세밀함)
- `/e2e`를 사용하여 통합 및 사용자 여정 테스트 수행
- `/code-review`를 사용하여 테스트 품질 검증

## 관련 에이전트

이 명령어는 다음 위치에 있는 `e2e-runner` 에이전트를 호출합니다:
`~/.claude/agents/e2e-runner.md`

## 퀵 명령어 (Quick Commands)

```bash
# 모든 E2E 테스트 실행
npx playwright test

# 특정 테스트 파일 실행
npx playwright test tests/e2e/markets/search.spec.ts

# 헤드 모드로 실행 (브라우저 확인)
npx playwright test --headed

# 테스트 디버깅
npx playwright test --debug

# 테스트 코드 생성
npx playwright codegen http://localhost:3000

# 보고서 보기
npx playwright show-report
```
