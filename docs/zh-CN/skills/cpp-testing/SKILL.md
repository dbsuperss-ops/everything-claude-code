---
name: cpp-testing
description: C++ 테스트를 작성, 업데이트 및 수정할 때, GoogleTest/CTest를 구성할 때, 실패하거나 불안정한 테스트를 진단할 때, 또는 코드 커버리지와 새니타이저(Sanitizer)를 추가할 때 사용합니다.
origin: ECC
---

# C++ 테스트 (에이전트 스킬)

GoogleTest/GoogleMock 및 CMake/CTest를 사용하는 현대적 C++(C++17/20) 환경의 에이전트 지향 테스트 워크플로우입니다.

## 적용 시점

* 새로운 C++ 테스트를 작성하거나 기존 테스트를 수정할 때
* C++ 컴포넌트를 위한 단위/통합 테스트 커버리지를 설계할 때
* 테스트 커버리지, CI 게이팅 또는 회귀 방지(Regression protection) 기능을 추가할 때
* 일관된 테스트 실행을 위해 CMake/CTest 워크플로우를 구성할 때
* 테스트 실패 원인이나 간헐적으로 발생하는 동작(Flaky behavior)을 조사할 때
* 메모리 에러 또는 경합 상태(Race condition) 진단을 위해 새니타이저를 활성화할 때

## 적용 제외 대상

* 테스트 수정 없이 새로운 제품 기능만 구현하는 경우
* 테스트 샘플링이나 실패 수정과 무관한 대규모 리팩토링
* 검증할 회귀 테스트가 없는 단순 성능 튜닝
* C++ 프로젝트가 아니거나 테스트와 무관한 작업

## 핵심 개념

* **TDD 사이클**: Red → Green → Refactor (테스트 먼저 작성, 최소한의 수정으로 통과, 이후 코드 정리).
* **격리(Isolation)**: 전역 상태 대신 의존성 주입(DI)과 모의 객체(Mock)를 우선적으로 사용하십시오.
* **디렉토리 구조**: `tests/unit`, `tests/integration`, `tests/testdata` 등으로 구분합니다.
* **Mock vs Fake**: 상호작용 검증에는 Mock을, 상태 기반 동작 구현에는 Fake를 사용하십시오.
* **CTest Discovery**: 안정적인 테스트 탐색을 위해 `gtest_discover_tests()`를 사용하십시오.
* **CI 신호**: 특정 부분 집합을 먼저 실행하고, 전체 스위트 실행 시 `--output-on-failure` 옵션을 활용하십시오.

## TDD 워크플로우

Red → Green → Refactor 사이클을 준수하십시오:

1. **RED**: 새로운 기능을 정의하는 실패하는 테스트를 작성합니다.
2. **GREEN**: 테스트가 통과할 수 있도록 최소한의 코드만 구현합니다.
3. **REFACTOR**: 테스트 통과 상태를 유지하면서 코드를 깨끗하게 정리합니다.

```cpp
// tests/add_test.cpp
#include <gtest/gtest.h>

int Add(int a, int b); // 제품 코드에서 제공됨

TEST(AddTest, AddsTwoNumbers) { // RED
  EXPECT_EQ(Add(2, 3), 5);
}

// src/add.cpp
int Add(int a, int b) { // GREEN
  return a + b;
}

// REFACTOR: 테스트 통과 후 이름 변경이나 로직 최적화 수행
```

## 코드 예시

### 기본 단위 테스트 (gtest)

```cpp
// tests/calculator_test.cpp
#include <gtest/gtest.h>

int Add(int a, int b); // 제품 코드에서 제공됨

TEST(CalculatorTest, AddsTwoNumbers) {
    EXPECT_EQ(Add(2, 3), 5);
}
```

### 테스트 픽스처 (gtest)

```cpp
// tests/user_store_test.cpp
// 의사코드 스텁: 실제 프로젝트의 UserStore/User 타입으로 대체하십시오.
#include <gtest/gtest.h>
#include <memory>
#include <optional>
#include <string>

struct User { std::string name; };
class UserStore {
public:
    explicit UserStore(std::string /*path*/) {}
    void Seed(std::initializer_list<User> /*users*/) {}
    std::optional<User> Find(const std::string &/*name*/) { return User{"alice"}; }
};

class UserStoreTest : public ::testing::Test {
protected:
    void SetUp() override {
        store = std::make_unique<UserStore>(":memory:");
        store->Seed({{"alice"}, {"bob"}});
    }

    std::unique_ptr<UserStore> store;
};

TEST_F(UserStoreTest, FindsExistingUser) {
    auto user = store->Find("alice");
    ASSERT_TRUE(user.has_value());
    EXPECT_EQ(user->name, "alice");
}
```

### 모의 객체 (gmock)

```cpp
// tests/notifier_test.cpp
#include <gmock/gmock.h>
#include <gtest/gtest.h>
#include <string>

class Notifier {
public:
    virtual ~Notifier() = default;
    virtual void Send(const std::string &message) = 0;
};

class MockNotifier : public Notifier {
public:
    MOCK_METHOD(void, Send, (const std::string &message), (override));
};

class Service {
public:
    explicit Service(Notifier &notifier) : notifier_(notifier) {}
    void Publish(const std::string &message) { notifier_.Send(message); }

private:
    Notifier &notifier_;
};

TEST(ServiceTest, SendsNotifications) {
    MockNotifier notifier;
    Service service(notifier);

    EXPECT_CALL(notifier, Send("hello")).Times(1);
    service.Publish("hello");
}
```

### CMake/CTest 빠른 시작

```cmake
# CMakeLists.txt (발췌)
cmake_minimum_required(VERSION 3.20)
project(example LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FetchContent)
# 프로젝트 정책에 따라 특정 버전을 고정하여 사용하십시오.
set(GTEST_VERSION v1.17.0)
FetchContent_Declare(
  googletest
  # Google Test 프레임워크 공식 저장소
  URL https://github.com/google/googletest/archive/refs/tags/${GTEST_VERSION}.zip
)
FetchContent_MakeAvailable(googletest)

add_executable(example_tests
  tests/calculator_test.cpp
  src/calculator.cpp
)
target_link_libraries(example_tests GTest::gtest GTest::gmock GTest::gtest_main)

enable_testing()
include(GoogleTest)
gtest_discover_tests(example_tests)
```

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j
ctest --test-dir build --output-on-failure
```

## 테스트 실행 방법

```bash
# 전체 테스트 실행
ctest --test-dir build --output-on-failure

# 특정 테스트만 실행 (정규식 사용)
ctest --test-dir build -R ClampTest
ctest --test-dir build -R "UserStoreTest.*" --output-on-failure

# 바이너리 직접 실행하여 필터링
./build/example_tests --gtest_filter=ClampTest.*
./build/example_tests --gtest_filter=UserStoreTest.FindsExistingUser
```

## 실패 디버깅

1. gtest filter를 사용하여 실패한 단일 테스트만 다시 실행합니다.
2. 실패한 단언문(Assertion) 주변에 스코프 로그를 추가합니다.
3. 새니타이저를 활성화하고 다시 실행합니다.
4. 근본 원인을 해결한 후 전체 테스트 스위트로 확장하여 검증합니다.

## 코드 커버리지

글로벌 플래그 대신 타겟 레벨의 설정을 선호하십시오.

```cmake
option(ENABLE_COVERAGE "커버리지 플래그 활성화" OFF)

if(ENABLE_COVERAGE)
  if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_options(example_tests PRIVATE --coverage)
    target_link_options(example_tests PRIVATE --coverage)
  elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(example_tests PRIVATE -fprofile-instr-generate -fcoverage-mapping)
    target_link_options(example_tests PRIVATE -fprofile-instr-generate)
  endif()
endif()
```

GCC + gcov + lcov 활용:
```bash
cmake -S . -B build-cov -DENABLE_COVERAGE=ON
cmake --build build-cov -j
ctest --test-dir build-cov
lcov --capture --directory build-cov --output-file coverage.info
lcov --remove coverage.info '/usr/*' --output-file coverage.info
genhtml coverage.info --output-directory coverage
```

Clang + llvm-cov 활용:
```bash
cmake -S . -B build-llvm -DENABLE_COVERAGE=ON -DCMAKE_CXX_COMPILER=clang++
cmake --build build-llvm -j
LLVM_PROFILE_FILE="build-llvm/default.profraw" ctest --test-dir build-llvm
llvm-profdata merge -sparse build-llvm/default.profraw -o build-llvm/default.profdata
llvm-cov report build-llvm/example_tests -instr-profile=build-llvm/default.profdata
```

## 새니타이저 (Sanitizers)

```cmake
option(ENABLE_ASAN "AddressSanitizer 활성화" OFF)
option(ENABLE_UBSAN "UndefinedBehaviorSanitizer 활성화" OFF)
option(ENABLE_TSAN "ThreadSanitizer 활성화" OFF)

if(ENABLE_ASAN)
  add_compile_options(-fsanitize=address -fno-omit-frame-pointer)
  add_link_options(-fsanitize=address)
endif()
if(ENABLE_UBSAN)
  add_compile_options(-fsanitize=undefined -fno-omit-frame-pointer)
  add_link_options(-fsanitize=undefined)
endif()
if(ENABLE_TSAN)
  add_compile_options(-fsanitize=thread)
  add_link_options(-fsanitize=thread)
endif()
```

## 불안정한(Flaky) 테스트 방지 가이드

* 동기화를 위해 절대로 `sleep`을 사용하지 마십시오. 조건 변수(Condition variable)나 래치(Latch)를 사용하십시오.
* 각 테스트마다 고유한 임시 디렉토리를 생성하고 실행 후 반드시 삭제하십시오.
* 단위 테스트에서 실제 시간, 네트워크 또는 파일 시스템에 의존하는 것을 피하십시오.
* 무작위 입력 값을 사용할 때는 결정론적인 시드(Deterministic seed)를 사용하십시오.

## 베스트 프랙티스

### 권장 사항 (DO)
* 테스트의 결정론(Determinism)과 격리를 유지하십시오.
* 전역 변수 대신 의존성 주입(DI)을 우선하십시오.
* 전제 조건 확인에는 `ASSERT_*`, 여러 항목 확인에는 `EXPECT_*`를 사용하십시오.
* CTest 라벨이나 디렉토리를 통해 단위 테스트와 통합 테스트를 분리하십시오.
* 메모리 에러 및 경합 상태 감지를 위해 CI에서 새니타이저를 실행하십시오.

### 금지 사항 (DON'T)
* 단위 테스트에서 실제 시간이나 네트워크에 의존하지 마십시오.
* 조건 변수를 쓸 수 있는 상황에서 `sleep`을 동기화 수단으로 쓰지 마십시오.
* 간단한 값 객체(Value objects)를 과도하게 모킹(Mocking)하지 마십시오.
* 중요하지 않은 로그 메시지를 검증하기 위해 취약한 문자열 매칭을 사용하지 마십시오.

### 흔한 함정
* **고정된 임시 경로 사용** → 테스트마다 고유한 임시 디렉토리를 생성하고 정리하십시오.
* **시스템 시각(Wall clock) 의존** → 클럭을 주입하거나 모의 시간 소스를 사용하십시오.
* **간헐적인 동시성 테스트 실패** → 조건 변수/래치와 타임아웃 대기를 활용하십시오.
* **숨겨진 전역 상태** → 픽스처에서 전역 상태를 리셋하거나 전역 변수를 제거하십시오.
* **과도한 모킹** → 상태 기반 동작에는 Fake 객체를 우선하고, 상호작용 검증에만 Mock을 쓰십시오.
* **새니타이저 미사용** → CI 과정에 ASan/UBSan/TSan 빌드를 추가하십시오.

## 기타 테스트 도구 (부록)

프로젝트에서 LLVM/libFuzzer나 속성 기반 테스트 라이브러리를 이미 지원하는 경우에만 사용하십시오.

* **libFuzzer**: 입출력이 적은 순수 함수 테스트에 적합합니다.
* **RapidCheck**: 변하지 않는 속성(Invariant) 검증을 위한 속성 기반 테스트 도구입니다.

최소 수준의 libFuzzer 테스트 구성 예시:
```cpp
#include <cstddef>
#include <cstdint>
#include <string>

extern "C" int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    std::string input(reinterpret_cast<const char *>(data), size);
    // ParseConfig(input); // 프로젝트 함수 호출
    return 0;
}
```

## GoogleTest의 대안
* **Catch2**: 헤더 전용 라이브러리로 표현력이 뛰어난 매처(Matcher)를 제공합니다.
* **doctest**: 가볍고 컴파일 속도가 매우 빠른 것이 장점입니다.
