---
name: cpp-testing
description: C++ 테스트 작성/업데이트/수정, GoogleTest/CTest 구성, 실패하거나 불안정한(Flaky) 테스트 진단, 또는 커버리지/새니타이저(Sanitizer) 추가 시에 사용하십시오.
origin: ECC
---

# C++ 테스트 (에이전트 스킬)

CMake/CTest와 GoogleTest/GoogleMock을 사용하는 현대적 C++(C++17/20)을 위한 에이전트 중심의 테스트 워크플로우입니다.

## 활성화 시점

- 새로운 C++ 테스트를 작성하거나 기존 테스트를 수정할 때
- C++ 컴포넌트를 위한 단위/통합 테스트 범위를 설계할 때
- 테스트 커버리지, CI 게이팅 또는 회귀 방지 로직을 추가할 때
- 일관된 실행을 위한 CMake/CTest 워크플로우를 구성할 때
- 테스트 실패 원인 또는 불안정한 동작(Flaky behavior)을 조사할 때
- 메모리 문제나 레이스 컨디션(Race condition) 진단을 위해 새니타이저를 활성화할 때

## 핵심 개념

- **TDD 루프**: Red(실패) → Green(성공) → Refactor(리팩토링) 순서를 따릅니다.
- **격리(Isolation)**: 전역 상태 대신 의존성 주입(Dependency Injection)과 페이크(Fakes) 객체 사용을 선호합니다.
- **테이스 레이아웃**: `tests/unit`, `tests/integration`, `tests/testdata` 구조로 구성합니다.
- **Mock vs Fake**: 상호작용 검증에는 Mock을, 상태 기반 동작 시뮬레이션에는 Fake를 사용합니다.
- **CTest 검색**: 안정적인 테스트 검색을 위해 `gtest_discover_tests()`를 사용합니다.

## TDD 워크플로우 (RED → GREEN → REFACTOR)

1. **RED**: 새로운 동작을 정의하고 실패하는 테스트를 먼저 작성합니다.
2. **GREEN**: 테스트를 통과시키기 위한 최소한의 변경 사항을 구현합니다.
3. **REFACTOR**: 테스트가 성공하는 상태를 유지하며 코드를 깔끔하게 정리합니다.

## 코드 예시

### 기본 단위 테스트 (gtest)
`EXPECT_EQ` 등을 사용하여 기댓값과 실제값을 비교합니다.

### 피복물 (Fixture, gtest)
`SetUp()`과 `TearDown()`을 사용하여 여러 테스트에서 공유할 환경을 설정합니다. `TEST_F`를 사용합니다.

### 모의 객체 (Mock, gmock)
`MOCK_METHOD`를 사용하여 인터페이스를 모킹하고, `EXPECT_CALL`로 특정 메서드가 호출되었는지 검증합니다.

## CMake/CTest 설정

`FetchContent`를 사용하여 GoogleTest를 가져오고, `gtest_discover_tests`를 통해 CTest가 테스트를 인식하도록 설정합니다.

```bash
# 실행 예시
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
ctest --test-dir build --output-on-failure
```

## 디버깅 및 커버리지

- 실패한 테스트를 `--gtest_filter` 옵션으로 개별 실행하여 조사하십시오.
- `ENABLE_COVERAGE` 옵션을 구성하여 gcov/lcov 또는 llvm-cov로 코드 커버리지를 측정하십시오.
- AddressSanitizer(ASan), UndefinedBehaviorSanitizer(UBSan), ThreadSanitizer(TSan)를 활용하여 메모리 에러와 경합 상태를 감지하십시오.

## 불안정한 테스트(Flaky Tests) 방지 가이드

- 동기화를 위해 `sleep`을 사용하지 마십시오. 대신 조건 변수(Condition variables)나 래치(Latches)를 사용하십시오.
- 테스트별로 고유한 임시 디렉토리를 생성하고 반드시 정리하십시오.
- 단위 테스트에서는 실제 시간, 네트워크, 파일 시스템 의존성을 피하십시오.

## 최선 관행 (Best Practices)

### 권장 사항 (DO)
- 테스트를 결정론적(Deterministic)이고 격리된 상태로 유지하십시오.
- 전역 변수보다 의존성 주입을 선호하십시오.
- 전제 조건 확인에는 `ASSERT_*`를, 단순 값 비교에는 `EXPECT_*`를 사용하십시오.
- CI 환경에서 새니타이저를 실행하여 메모리 문제를 조기에 발견하십시오.

### 지양 사항 (DON'T)
- 단위 테스트에서 실제 네트워크나 실시간 시간에 의존하지 마십시오.
- 조건 변수로 해결 가능한 상황에서 `sleep`을 사용하지 마십시오.
- 단순한 값 객체(Value objects)를 과도하게 모킹하지 마십시오.

**기억하십시오**: 테스트는 코드의 품질을 보장하는 동시에 살아있는 문서 역할을 합니다. 테스트 코드를 명확하고 견고하게 유지하여 신뢰할 수 있는 시스템을 구축하십시오.
    
