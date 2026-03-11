---
name: iterative-retrieval
description: 서브 에이전트의 컨텍스트 부족 문제를 해결하기 위해 컨텍스트 검색을 점진적으로 정교화하는 패턴입니다.
origin: ECC
---

# 반복적 검색 패턴 (Iterative Retrieval Pattern)

멀티 에이전트 워크플로우에서 서브 에이전트가 작업을 시작하기 전까지는 어떤 컨텍스트가 필요한지 정확히 알지 못하는 "컨텍스트 문제"를 해결합니다.

## 활성화 시점

- 사전에 예측하기 어려운 코드베이스 컨텍스트가 필요한 서브 에이전트를 생성할 때
- 컨텍스트가 점진적으로 정교해지는 멀티 에이전트 워크플로우를 구축할 때
- 에이전트 작업 중 "컨텍스트 너무 큼" 또는 "컨텍스트 부족" 오류가 발생할 때
- 코드 탐색을 위한 RAG(검색 증강 생성) 스타일의 검색 파이프라인을 설계할 때
- 에이전트 오케스트레이션에서 토큰 사용량을 최적화할 때

## 문제 상황

서브 에이전트는 제한된 컨텍스트를 가지고 생성됩니다. 에이전트는 다음을 알지 못합니다:
- 관련 코드가 어느 파일에 있는지
- 코드베이스에 어떤 패턴이 존재하는지
- 프로젝트에서 어떤 용어를 사용하는지

표준적인 접근 방식은 실패하기 쉽습니다:
- **전부 보내기**: 컨텍스트 제한을 초과합니다.
- **아무것도 안 보내기**: 에이전트가 중요한 정보를 얻지 못합니다.
- **필요할 것 같은 것 추측하기**: 자주 빗나갑니다.

## 해결책: 반복적 검색

컨텍스트를 점진적으로 정교화하는 4단계 루프입니다:

```
┌─────────────────────────────────────────────┐
│                                             │
│   ┌──────────┐      ┌──────────┐            │
│   │   파견   │─────▶│   평가   │            │
│   │ (Dispatch)│      │(Evaluate)│            │
│   └──────────┘      └──────────┘            │
│        ▲                  │                 │
│        │                  ▼                 │
│   ┌──────────┐      ┌──────────┐            │
│   │   루프   │◀─────│   정제   │            │
│   │  (Loop)  │      │ (Refine) │            │
│   └──────────┘      └──────────┘            │
│                                             │
│      최대 3회 반복 후 다음 단계 진행           │
└─────────────────────────────────────────────┘
```

### 단계 1: 파견 (DISPATCH)

후보 파일을 모으기 위한 초기 광범위 쿼리입니다:

```javascript
// 높은 수준의 의도로 시작
const initialQuery = {
  patterns: ['src/**/*.ts', 'lib/**/*.ts'],
  keywords: ['authentication', 'user', 'session'],
  excludes: ['*.test.ts', '*.spec.ts']
};

// 검색 에이전트에게 파견
const candidates = await retrieveFiles(initialQuery);
```

### 단계 2: 평가 (EVALUATE)

검색된 내용의 관련성을 평가합니다:

```javascript
function evaluateRelevance(files, task) {
  return files.map(file => ({
    path: file.path,
    relevance: scoreRelevance(file.content, task),
    reason: explainRelevance(file.content, task),
    missingContext: identifyGaps(file.content, task)
  }));
}
```

점수 기준:
- **높음 (0.8-1.0)**: 대상 기능을 직접 구현함
- **중간 (0.5-0.7)**: 관련 패턴이나 타입을 포함함
- **낮음 (0.2-0.4)**: 부수적으로 관련됨
- **없음 (0-0.2)**: 관련 없음, 제외 대상

### 단계 3: 정제 (REFINE)

평가 결과를 바탕으로 검색 기준을 업데이트합니다:

```javascript
function refineQuery(evaluation, previousQuery) {
  return {
    // 관련성 높은 파일에서 발견된 새로운 패턴 추가
    patterns: [...previousQuery.patterns, ...extractPatterns(evaluation)],

    // 코드베이스에서 발견된 용어 추가
    keywords: [...previousQuery.keywords, ...extractKeywords(evaluation)],

    // 관련 없음으로 확인된 경로 제외
    excludes: [...previousQuery.excludes, ...evaluation
      .filter(e => e.relevance < 0.2)
      .map(e => e.path)
    ],

    // 특정 부족한 부분(Gap) 타겟팅
    focusAreas: evaluation
      .flatMap(e => e.missingContext)
      .filter(unique)
  };
}
```

### 단계 4: 루프 (LOOP)

정제된 기준으로 최대 3회까지 반복합니다. 관련성 높은 파일을 충분히 확보하면 조기에 종료합니다.

## 실제 사례

### 사례 1: 버그 수정 컨텍스트

```
작업: "인증 토큰 만료 버그 수정"

1주기:
  파견: src/** 에서 "token", "auth", "expiry" 검색
  평가: auth.ts (0.9), tokens.ts (0.8), user.ts (0.3) 발견
  정제: "refresh", "jwt" 키워드 추가, user.ts 제외

2주기:
  파견: 정제된 용어로 검색
  평가: session-manager.ts (0.95), jwt-utils.ts (0.85) 발견
  정제: 충분한 컨텍스트 확보 (관련성 높은 파일 다수)

결과: auth.ts, tokens.ts, session-manager.ts, jwt-utils.ts
```

## 최선 관행 (Best Practices)

1. **넓게 시작하고 점진적으로 좁히십시오** - 초기 쿼리를 너무 상세하게 잡지 마십시오.
2. **코드베이스 용어를 학습하십시오** - 첫 번째 주기에서 명명 규칙이 드러나는 경우가 많습니다.
3. **무엇이 부족한지 추적하십시오** - 명확한 간극(Gap) 식별이 정교한 검색을 이끕니다.
4. **"충분함" 수준에서 멈추십시오** - 관련성 높은 3개의 파일이 평범한 10개보다 낫습니다.
5. **확신을 가지고 제외하십시오** - 관련성 낮은 파일이 나중에 관련 있어질 확률은 적습니다.
