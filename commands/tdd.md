---
이름: tdd
설명: 테스트 주도 개발(TDD) 워크플로우를 강제합니다. 인터페이스를 설계하고, 테스트를 **가장 먼저** 생성한 후, 이를 통과하기 위한 최소한의 코드를 구현합니다. 80% 이상의 커버리지를 보장합니다.
---

# TDD 명령어 (TDD Command)

이 명령어는 테스트 주도 개발 방법론을 강제하기 위해 **tdd-guide** 에이전트를 호출합니다.

## 주요 기능

1. **인터페이스 설계 (Scaffold)** - 타입/인터페이스를 먼저 정의합니다.
2. **테스트 우선 생성 (RED)** - 실패하는 테스트를 먼저 작성합니다.
3. **최소 코드 구현 (GREEN)** - 테스트 통과를 위한 최소한의 코드만 작성합니다.
4. **리팩토링 (REFACTOR)** - 테스트 통과 상태를 유지하며 코드를 개선합니다.
5. **커버리지 검증** - 80% 이상의 테스트 커버리지를 확보합니다.

## 사용 시점

다음과 같은 경우 `/tdd`를 사용하십시오:
- 새로운 기능을 구현할 때
- 새로운 함수/컴포넌트를 추가할 때
- 버그를 수정할 때 (버그를 재현하는 테스트를 먼저 작성)
- 기존 코드를 리팩토링할 때
- 중요한 비즈니스 로직을 구축할 때

## 동작 방식

tdd-guide 에이전트는 다음 작업을 수행합니다:

1. 입력/출력을 위한 **인터페이스 정의**
2. **실패할 테스트 작성** (코드가 아직 없으므로)
3. **테스트 실행** 및 의도한 사유로 실패하는지 확인
4. 테스트 통과를 위한 **최소한의 구현** 작성
5. **테스트 실행** 및 통과 여부 확인
6. 테스트 통과 상태를 유지하며 코드 **리팩토링**
7. **커버리지 확인** 및 80% 미만인 경우 테스트 추가

## TDD 사이클

```
RED → GREEN → REFACTOR → REPEAT

RED:      실패하는 테스트 작성
GREEN:    통과를 위한 최소한의 코드 작성
REFACTOR: 코드를 개선하되 테스트 통과 상태 유지
REPEAT:   다음 기능/시나리오로 반복
```

## 사용 예시

```
사용자: /tdd 마켓 유동성 점수를 계산하는 함수가 필요해

에이전트 (tdd-guide):
# TDD 세션: 마켓 유동성 점수 계산기

## 1단계: 인터페이스 정의 (SCAFFOLD)

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: 구현 예정
  throw new Error('Not implemented')
}
```

## 2단계: 실패하는 테스트 작성 (RED)

```typescript
// lib/liquidity.test.ts
import { calculateLiquidityScore } from './liquidity'

describe('calculateLiquidityScore', () => {
  it('유동성이 풍부한 마켓에 대해 높은 점수를 반환해야 함', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('유동성이 부족한 마켓에 대해 낮은 점수를 반환해야 함', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000) // 1일 전
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeLessThan(30)
    expect(score).toBeGreaterThanOrEqual(0)
  })

  it('엣지 케이스: 거래량이 0인 경우를 처리해야 함', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBe(0)
  })
})
```

## 3단계: 테스트 실행 - 실패 확인

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ 유동성이 풍부한 마켓에 대해 높은 점수를 반환해야 함 (2 ms)
    Error: Not implemented

1 test failed, 0 passed
```

✅ 예상대로 테스트가 실패했습니다. 구현을 시작합니다.

## 4단계: 최소한의 코드 구현 (GREEN)

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // 거래량 0인 엣지 케이스 처리
  if (market.totalVolume === 0) {
    return 0
  }

  // 구성 항목별 점수 계산 (0-100 스케일)
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // 최근 활동 보너스
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // 가중 평균 계산
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // 0-100 사이로 고정
}
```

## 5단계: 테스트 실행 - 성공 확인

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ 유동성이 풍부한 마켓에 대해 높은 점수를 반환해야 함 (3 ms)
  ✓ 유동성이 부족한 마켓에 대해 낮은 점수를 반환해야 함 (2 ms)
  ✓ 엣지 케이스: 거래량이 0인 경우를 처리해야 함 (1 ms)

3 tests passed
```

✅ 모든 테스트를 통과했습니다!

## 6단계: 리팩토링 (IMPROVE)

```typescript
// lib/liquidity.ts - 상수 사용 및 가독성 개선을 위한 리팩토링
const WEIGHTS = {
  VOLUME: 0.4,
  SPREAD: 0.3,
  TRADERS: 0.2,
  RECENCY: 0.1,
} as const

const SCALE_FACTORS = {
  VOLUME: 1000,
  SPREAD: 1000,
  TRADERS: 10,
  RECENCY_PENALTY: 10,
} as const

function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}

export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0

  const volumeScore = Math.min(market.totalVolume / SCALE_FACTORS.VOLUME, 100)
  const spreadScore = clamp(100 - (market.bidAskSpread * SCALE_FACTORS.SPREAD), 0, 100)
  const traderScore = Math.min(market.activeTraders / SCALE_FACTORS.TRADERS, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = clamp(100 - (hoursSinceLastTrade * SCALE_FACTORS.RECENCY_PENALTY), 0, 100)

  const weightedScore =
    volumeScore * WEIGHTS.VOLUME +
    spreadScore * WEIGHTS.SPREAD +
    traderScore * WEIGHTS.TRADERS +
    recencyScore * WEIGHTS.RECENCY

  return clamp(weightedScore, 0, 100)
}
```

## 7단계: 테스트 재실행 및 성공 확인

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ 유동성이 풍부한 마켓에 대해 높은 점수를 반환해야 함 (3 ms)
  ✓ 유동성이 부족한 마켓에 대해 낮은 점수를 반환해야 함 (2 ms)
  ✓ 엣지 케이스: 거래량이 0인 경우를 처리해야 함 (1 ms)

3 tests passed
```

✅ 리팩토링 완료, 여전히 모든 테스트를 통과합니다!

## 8단계: 커버리지 확인

```bash
npm test -- --coverage lib/liquidity.test.ts

파일           | % 구문 | % 분기 | % 함수 | % 라인
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

커버리지: 100% ✅ (목표: 80%)
```

✅ TDD 세션 완료!
```

## TDD 최선 관행

**권장 사항 (DO):**
- ✅ 구현 전 반드시 테스트를 먼저 작성하십시오.
- ✅ 구현 전 테스트를 실행하여 실패(RED)하는지 확인하십시오.
- ✅ 테스트 통과를 위한 최소한의 코드만 작성하십시오.
- ✅ 테스트 통과 후에만 리팩토링을 수행하십시오.
- ✅ 엣지 케이스와 에러 시나리오를 추가하십시오.
- ✅ 80% 이상의 커버리지를 목표로 하십시오 (핵심 코드는 100%).

**지양 사항 (DON'T):**
- ❌ 테스트보다 구현을 먼저 하지 마십시오.
- ❌ 변경 사항마다 테스트를 실행하는 과정을 건너뛰지 마십시오.
- ❌ 한 번에 너무 많은 코드를 작성하지 마십시오.
- ❌ 실패하는 테스트를 무시하지 마십시오.
- ❌ 구현 상세를 테스트하지 마십시오 (동작을 테스트하십시오).
- ❌ 모든 것을 모킹(Mock)하지 마십시오 (통합 테스트를 선호하십시오).

## 포함해야 할 테스트 유형

**단위 테스트 (Unit Tests)** (함수 수준):
- 정상적인 경로 시나리오
- 엣지 케이스 (empty, null, 최대값 등)
- 에러 조건
- 경계값(Boundary values)

**통합 테스트 (Integration Tests)** (컴포넌트 수준):
- API 엔드포인트
- 데이터베이스 작업
- 외부 서비스 호출
- 훅(hooks)이 포함된 React 컴포넌트

**E2E 테스트** (`/e2e` 명령어 사용):
- 핵심 사용자 흐름
- 다단계 프로세스
- 풀스택 통합

## 커버리지 요구사항

- 모든 코드에 대해 **최소 80%**
- 다음 항목은 **100% 필수**:
  - 금융 관련 계산
  - 인증 로직
  - 보안에 민감한 코드
  - 핵심 비즈니스 로직

## 중요 사항

**필수**: 테스트는 반드시 구현 전에 작성되어야 합니다. TDD 사이클은 다음과 같습니다:

1. **RED** - 실패하는 테스트 작성
2. **GREEN** - 통과하도록 구현
3. **REFACTOR** - 코드 개선

절대 RED 단계를 건너뛰지 마십시오. 테스트 전에 코드를 작성하지 마십시오.

## 다른 명령어와의 통합

- 구현할 내용을 파악하기 위해 `/plan`을 먼저 사용하십시오.
- 테스트와 함께 구현하기 위해 `/tdd`를 사용하십시오.
- 빌드 에러 발생 시 `/build-fix`를 사용하십시오.
- 구현 완료 후 `/code-review`를 사용하여 리뷰하십시오.
- 커버리지 확인을 위해 `/test-coverage`를 사용하십시오.

## 관련 에이전트 및 스킬

이 명령어는 다음 경로에 있는 `tdd-guide` 에이전트를 호출합니다:
`~/.claude/agents/tdd-guide.md`

또한 다음 경로의 `tdd-workflow` 스킬을 참조할 수 있습니다:
`~/.claude/skills/tdd-workflow/`
