---
name: e2e-testing
description: Playwright E2E 테스트 패턴, 페이지 오브젝트 모델(POM), 설정, CI/CD 통합, 산출물(Artifact) 관리 및 불안정한(Flaky) 테스트 대응 전략 가이드입니다.
origin: ECC
---

# E2E 테스트 패턴 (E2E Testing Patterns)

안정적이고 빠르며 유지보수가 용이한 E2E(End-to-End) 테스트 스위트 구축을 위한 Playwright 활용 패턴입니다.

## 테스트 파일 구성

- **tests/e2e/**: 기능별 테스트 파일 (.spec.ts)
- **fixtures/**: 인증 정보나 공통 테스트 데이터
- **playwright.config.ts**: 전체 테스트 환경 설정

## 페이지 오브젝트 모델 (Page Object Model, POM)

UI 요소를 캡슐화하여 테스트 코드의 가독성을 높이고 중복을 줄입니다.

```typescript
// pages/ItemsPage.ts 예시
export class ItemsPage {
  readonly page: Page
  readonly searchInput: Locator

  constructor(page: Page) {
    this.page = page
    this.searchInput = page.locator('[data-testid="search-input"]')
  }

  async search(query: string) {
    await this.searchInput.fill(query)
    // 네트워크 응답 대기 및 로드 상태 확인
    await this.page.waitForResponse(resp => resp.url().includes('/api/search'))
    await this.page.waitForLoadState('networkidle')
  }
}
```

## Playwright 설정 (Configuration)

- **fullyParallel**: 테스트를 병렬로 실행하여 속도 향상
- **retries**: CI 환경에서 실패 시 재시도 횟수 설정
- **reporter**: HTML, JUnit, JSON 등 다양한 보고서 형식 지정
- **use**: 기본 URL, 트레이스(Trace) 저장 조건, 스크린샷 및 비디오 녹화 조건 설정

## 불안정한 테스트(Flaky Test) 대응

### 격리 및 스킵
- `test.fixme()`: 이슈 해결 전까지 잠정 중단
- `test.skip()`: 특정 조건(예: CI 환경 전용)에서 테스트 건너뛰기

### 원인 파악 및 해결
- **경합 상태(Race condition)**: 단순 `click()` 대신 자동 대기 기능이 있는 로케이터(`locator().click()`)를 사용하십시오.
- **네트워크 타이밍**: `waitForTimeout(5000)`과 같은 고정 대기 대신 특정 API 응답(`waitForResponse`)을 기다리십시오.
- **애니메이션 타이밍**: 요소가 완전히 나타나거나 안정화될 때까지 `waitFor({ state: 'visible' })` 등으로 대기하십시오.

## 산출물 관리 (Artifact Management)

- **스크린샷**: 특정 시점이나 실패 시 자동 저장
- **트레이스(Trace)**: 테스트 실행 과정을 타임라인별로 기록하여 사후 디버깅에 활용
- **비디오**: 실패한 테스트의 실행 과정을 녹화(webm 형식)

## CI/CD 통합 (GitHub Actions)

가상 환경(Ubuntu 등)에서 Playwright 브라우저를 설치하고 테스트를 실행한 후, 결과 보고서를 업로드하도록 파이프라인을 구성합니다.

## 지갑 / Web3 테스트

지갑 프로바이더(`window.ethereum`)를 모킹하여 실제 지갑 설치 없이 서명이나 계정 연결 과정을 테스트할 수 있습니다.

**기억하십시오**: E2E 테스트는 실제 사용자 경험과 가장 가깝지만 비용이 많이 듭니다. 핵심 사용자 흐름(Happy path)과 크리티컬한 에러 케이스 위주로 구성하고, 모든 것을 E2E로 테스트하려 하지 마십시오.
    
