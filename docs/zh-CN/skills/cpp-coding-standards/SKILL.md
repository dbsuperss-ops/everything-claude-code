---
name: cpp-coding-standards
description: C++ 핵심 가이드라인(isocpp.github.io)을 기반으로 한 C++ 코딩 표준입니다. 현대적이고 안전하며 관용적인 C++ 작성을 위해 코드 작성, 리뷰 또는 리팩토링 시 사용합니다.
origin: ECC
---

# C++ 코딩 표준 (C++ Core Guidelines 기반)

[C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)에서 파생된 현대적 C++(C++17/20/23) 종합 코딩 표준입니다. 타입 안정성, 리소스 안전성, 불변성 및 명확성을 강제합니다.

## 적용 시점

* 새로운 C++ 코드(클래스, 함수, 템플릿 등)를 작성할 때
* 기존 C++ 코드를 리뷰하거나 리팩토링할 때
* C++ 프로젝트 내에서 아키텍처 결정을 내릴 때
* C++ 코드베이스 전체에 일관된 스타일을 적용할 때
* 언어 기능 선택 시 (예: `enum` vs `enum class`, 원시 포인터 vs 스마트 포인터 등)

### 적용 제외 대상

* C++이 아닌 프로젝트
* 현대적 C++ 이식성이 없는 레거시 C 코드베이스
* 가이드라인이 하드웨어 제약과 충돌하는 임베디드/베어메탈 환경 (선택적 적용 권장)

## 핵심 원칙

가이드라인 전반에 흐르는 핵심 기반 원칙들입니다:

1. **RAII 생활화** (P.8, R.1, E.6, CP.20): 리소스의 수명을 객체의 수명에 바인딩하십시오.
2. **기본적으로 불변성(Immutability) 유지** (P.10, Con.1-5, ES.25): `const`/`constexpr`로 시작하고, 변경 가능성(mutability)은 예외적인 상황에만 허용하십시오.
3. **타입 안정성** (P.4, I.4, ES.46-49, Enum.3): 타입 시스템을 활용하여 컴파일 타임에 에러를 방지하십시오.
4. **의도 표현** (P.3, F.1, NL.1-2, T.10): 이름, 타입, 개념이 코드의 목적을 명확히 전달해야 합니다.
5. **복잡성 최소화** (F.2-3, ES.5, Per.4-5): 단순한 코드가 곧 올바른 코드입니다.
6. **포인터보다 값 의미론(Value Semantics) 선호** (C.10, R.3-5, F.20, CP.31): 값 반환(Return by value)과 지역 객체 활용을 우선하십시오.

## 철학 및 인터페이스 (P.*, I.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **P.1** | 아이디어를 코드에 직접적으로 표현하십시오. |
| **P.3** | 의도를 명확히 표현하십시오. |
| **P.4** | 프로그램은 가급적 정적 타입 안전성을 갖춰야 합니다. |
| **P.5** | 런타임 체크보다 컴파일 타임 체크를 우선하십시오. |
| **P.8** | 어떤 리소스도 누출(Leak)하지 마십시오. |
| **P.10** | 가변 데이터보다 불변 데이터를 선호하십시오. |
| **I.1** | 인터페이스를 명시적으로 만드십시오. |
| **I.2** | const가 아닌 전역 변수를 피하십시오. |
| **I.4** | 인터페이스를 정밀하고 강한 타입(Strongly typed)으로 만드십시오. |
| **I.11** | 원시 포인터나 참조를 통해 소유권을 직접적으로 이전하지 마십시오. |
| **I.23** | 함수 파라미터 개수를 적게 유지하십시오. |

### 권장 사례 (DO)

```cpp
// P.10 + I.4: 불변성을 갖춘 강한 타입의 인터페이스
struct Temperature {
    double kelvin;
};

Temperature boil(const Temperature& water);
```

### 금지 사례 (DON'T)

```cpp
// 나쁜 인터페이스: 소유권 불분명, 단위 불분명
double boil(double* temp);

// const가 아닌 전역 변수 (I.2 위반)
int g_counter = 0;
```

## 함수 (F.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **F.1** | 의미 있는 작업은 잘 명명된 함수로 묶으십시오. |
| **F.2** | 함수는 하나의 논리적 작업만 수행해야 합니다. |
| **F.3** | 함수를 짧고 간단하게 유지하십시오. |
| **F.4** | 컴파일 타임에 평가될 수 있는 함수는 `constexpr`로 선언하십시오. |
| **F.6** | 절대 예외를 던지지 않는 함수는 `noexcept`로 선언하십시오. |
| **F.8** | 순수 함수(Pure functions)를 선호하십시오. |
| **F.16** | 입력 파라미터의 경우, 복사 비용이 적은 타입은 값으로, 나머지는 `const&`로 전달하십시오. |
| **F.20** | 출력값에 대해서는 출력용 파라미터보다 반환값을 선호하십시오. |
| **F.21** | 여러 값을 반환할 때는 구조체를 활용하십시오. |
| **F.43** | 지역 객체에 대한 포인터나 참조를 절대 반환하지 마십시오. |

### 파라미터 전달

```cpp
// F.16: 가벼운 타입은 값으로, 무거운 타입은 const&로
void print(int x);                           // 가벼움: 값 전달
void analyze(const std::string& data);       // 무거움: const& 전달
void transform(std::string s);               // 이동 대상: 값 전달 (이동됨)

// F.20 + F.21: 출력 파라미터보다 반환값 선호
struct ParseResult {
    std::string token;
    int position;
};

ParseResult parse(std::string_view input);   // 좋음: 구조체 반환

// 나쁨: 출력용 파라미터 사용
void parse(std::string_view input,
           std::string& token, int& pos);    // 가급적 피하십시오
```

### 순수 함수 및 constexpr

```cpp
// F.4 + F.8: 가능하면 순수 함수 및 constexpr 적용
constexpr int factorial(int n) noexcept {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

static_assert(factorial(5) == 120);
```

### 안티 패턴
* 함수에서 `T&&` 반환 (F.45)
* `va_arg` / C 스타일 가변 인자 사용 (F.55)
* 다른 스레드로 전달되는 lambda에서 참조로 캡처 (F.53)
* `const T`를 반환하여 이동 의미론(Move semantics) 방해 (F.49)

## 클래스 및 클래스 계층 구조 (C.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **C.2** | 불변성(Invariant)이 필요하면 `class`를, 멤버가 독립적이면 `struct`를 사용하십시오. |
| **C.9** | 멤버의 노출을 최소화하십시오. |
| **C.20** | 컴파일러가 기본 로직을 생성할 수 있다면 직접 정의하지 마십시오(Rule of Zero). |
| **C.21** | 복사/이동/소멸자 중 하나라도 정의하거나 `=delete`하면 나머지도 모두 처리하십시오(Rule of Five). |
| **C.35** | 베이스 클래스 소멸자: public virtual이거나 protected non-virtual이어야 합니다. |
| **C.41** | 생성자는 완전히 초기화된 객체를 만들어야 합니다. |
| **C.46** | 파라미터가 하나인 생성자는 `explicit`으로 선언하십시오. |
| **C.67** | 다형성 클래스는 public 복사/이동을 금지해야 합니다. |
| **C.128** | 가상 함수 명시: `virtual`, `override`, `final` 중 정확히 하나만 사용하십시오. |

### Rule of Zero (영의 규칙)

```cpp
// C.20: 컴파일러가 특수 멤버 함수를 생성하게 둠
struct Employee {
    std::string name;
    std::string department;
    int id;
    // 소멸자, 복사/이동 생성자, 대입 연산자를 직접 작성할 필요 없음
};
```

### Rule of Five (다섯의 규칙)

```cpp
// C.21: 리소스를 직접 관리해야 한다면 5개 모두 정의하십시오.
class Buffer {
public:
    explicit Buffer(std::size_t size)
        : data_(std::make_unique<char[]>(size)), size_(size) {}

    ~Buffer() = default;

    Buffer(const Buffer& other)
        : data_(std::make_unique<char[]>(other.size_)), size_(other.size_) {
        std::copy_n(other.data_.get(), size_, data_.get());
    }

    Buffer& operator=(const Buffer& other) {
        if (this != &other) {
            auto new_data = std::make_unique<char[]>(other.size_);
            std::copy_n(other.data_.get(), other.size_, new_data.get());
            data_ = std::move(new_data);
            size_ = other.size_;
        }
        return *this;
    }

    Buffer(Buffer&&) noexcept = default;
    Buffer& operator=(Buffer&&) noexcept = default;

private:
    std::unique_ptr<char[]> data_;
    std::size_t size_;
};
```

### 클래스 계층 구조

```cpp
// C.35 + C.128: 가상 소멸자 정의, override 사용
class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;  // C.121: 순수 인터페이스
};

class Circle : public Shape {
public:
    explicit Circle(double r) : radius_(r) {}
    double area() const override { return 3.14159 * radius_ * radius_; }

private:
    double radius_;
};
```

### 안티 패턴
* 생성자/소멸자 내에서 가상 함수 호출 (C.82)
* 비트리비얼(Non-trivial) 타입에 `memset`/`memcpy` 사용 (C.90)
* 가상 함수와 오버라이딩된 함수에 서로 다른 기본 매개변수 사용 (C.140)
* 데이터 멤버를 `const`나 참조로 선언하여 이동/복사 방해 (C.12)

## 리소스 관리 (R.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **R.1** | RAII를 사용하여 리소스를 자동으로 관리하십시오. |
| **R.3** | 원시 포인터(`T*`)는 비소유(Non-owning)를 의미합니다. |
| **R.5** | 지역(Scoped) 객체를 선호하십시오. 불필요하게 힙 할당을 하지 마십시오. |
| **R.10** | `malloc()`/`free()`를 피하십시오. |
| **R.11** | 명시적인 `new`와 `delete` 호출을 피하십시오. |
| **R.20** | 소유권을 표현할 때는 `unique_ptr` 또는 `shared_ptr`를 사용하십시오. |
| **R.21** | 소유권을 공유해야 하는 경우가 아니면 `shared_ptr`보다 `unique_ptr`를 사용하십시오. |
| **R.22** | `shared_ptr` 생성 시 `make_shared()`를 사용하십시오. |

### 스마트 포인터 사용

```cpp
// R.11 + R.20 + R.21: 스마트 포인터를 활용한 RAII
auto widget = std::make_unique<Widget>("config");  // 단독 소유
auto cache  = std::make_shared<Cache>(1024);        // 공유 소유

// R.3: 원시 포인터 = 비소유 관찰자
void render(const Widget* w) {  // w를 소유하지 않음
    if (w) w->draw();
}

render(widget.get());
```

### RAII 패턴

```cpp
// R.1: 리소스 획득은 초기화임(RAII)
class FileHandle {
public:
    explicit FileHandle(const std::string& path)
        : handle_(std::fopen(path.c_str(), "r")) {
        if (!handle_) throw std::runtime_error("파일 열기 실패: " + path);
    }

    ~FileHandle() {
        if (handle_) std::fclose(handle_);
    }

    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
    FileHandle(FileHandle&& other) noexcept
        : handle_(std::exchange(other.handle_, nullptr)) {}
    FileHandle& operator=(FileHandle&& other) noexcept {
        if (this != &other) {
            if (handle_) std::fclose(handle_);
            handle_ = std::exchange(other.handle_, nullptr);
        }
        return *this;
    }

private:
    std::FILE* handle_;
};
```

### 안티 패턴
* 생 포인터에 대한 `new`/`delete` 직접 노출 (R.11)
* C++ 코드 내의 `malloc()`/`free()` (R.10)
* 단일 표현식 내에서 여러 개의 리소스 할당 (R.13 -- 예외 안전성 위험)
* `unique_ptr`로 충분한 상황에 `shared_ptr` 사용 (R.21)

## 표현식 및 문장 (ES.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **ES.5** | 변수의 범위를 작게 유지하십시오. |
| **ES.20** | 항상 객체를 초기화하십시오. |
| **ES.23** | `{}` 중괄호 초기화 문법을 선호하십시오. |
| **ES.25** | 변경할 의도가 없다면 객체를 `const` 또는 `constexpr`로 선언하십시오. |
| **ES.28** | 복잡한 `const` 변수 초기화에는 람다(Lambda)를 활용하십시오. |
| **ES.45** | 매직 상수를 피하고 명명된 상수를 사용하십시오. |
| **ES.46** | 정보 손실이 발생하는 산술 변환을 피하십시오. |
| **ES.47** | `0`이나 `NULL` 대신 `nullptr`를 사용하십시오. |
| **ES.48** | 타입 캐스팅을 가급적 피하십시오. |
| **ES.50** | `const`를 임의로 제거하지 마십시오(const_cast 지양). |

### 초기화

```cpp
// ES.20 + ES.23 + ES.25: 항상 초기화, {} 선호, 기본 const 적용
const int max_retries{3};
const std::string name{"widget"};
const std::vector<int> primes{2, 3, 5, 7, 11};

// ES.28: 복잡한 const 초기화를 위한 람다 활용
const auto config = [&] {
    Config c;
    c.timeout = std::chrono::seconds{30};
    c.retries = max_retries;
    c.verbose = debug_mode;
    return c;
}();
```

### 안티 패턴
* 초기화되지 않은 변수 (ES.20)
* 포인터 값으로 `0` 또는 `NULL` 사용 (ES.47 -- `nullptr` 사용 권장)
* C 스타일 타입 캐스팅 (ES.48 -- `static_cast`, `const_cast` 등 고수준 캐스트 사용 권장)
* `const` 제거 (ES.50)
* 이름 없는 매직 넘버 (ES.45)
* 부호 있는 정수와 부호 없는 정수의 혼합 연산 (ES.100)
* 중첩된 범위에서 동일한 이름 재사용 (ES.12)

## 에러 처리 (E.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **E.1** | 설계 초기 단계에서 에러 처리 전략을 수립하십시오. |
| **E.2** | 함수가 할당된 작업을 완수할 수 없을 때 예외를 던지십시오. |
| **E.6** | 리소스 누수 방지를 위해 RAII를 사용하십시오. |
| **E.12** | 예외를 던지는 것이 불가능하거나 부적절한 경우 `noexcept`를 사용하십시오. |
| **E.14** | 에러 표현을 위해 명확하게 설계된 사용자 정의 타입을 예외로 사용하십시오. |
| **E.15** | 값으로 던지고(Throw by value), 참조로 받으십시오(Catch by reference). |
| **E.16** | 소멸자, 리소스 해제, swap 함수는 절대 실패해서는 안 됩니다. |
| **E.17** | 모든 함수에서 모든 예외를 잡으려고 시도하지 마십시오. |

### 예외 계층 구조

```cpp
// E.14 + E.15: 커스텀 예외 타입 활용, 값으로 던지고 참조로 잡기
class AppError : public std::runtime_error {
public:
    using std::runtime_error::runtime_error;
};

class NetworkError : public AppError {
public:
    NetworkError(const std::string& msg, int code)
        : AppError(msg), status_code(code) {}
    int status_code;
};

void fetch_data(const std::string& url) {
    // E.2: 실패 신호를 위해 예외 발생
    throw NetworkError("연결 거부됨", 503);
}

void run() {
    try {
        fetch_data("https://api.example.com");
    } catch (const NetworkError& e) {
        log_error(e.what(), e.status_code);
    } catch (const AppError& e) {
        log_error(e.what());
    }
    // E.17: 모든 것을 잡지 말고 예상치 못한 에러는 상위로 전파되게 둠
}
```

### 안티 패턴
* `int`나 문자열 리터럴 같은 내장 타입을 직접 던지는 행위 (E.14)
* 값으로 예외 캡처(Catch by value) -- 슬라이싱 위험 있음 (E.15)
* 에러를 조용히 무시하는 비어있는 catch 블록
* 제어 흐름(Control flow)을 위해 예외 사용 (E.3)
* `errno` 같은 전역 상태 기반의 에러 처리 (E.28)

## 상수 및 불변성 (Con.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **Con.1** | 기본적으로 객체를 불변(Immutable)으로 만드십시오. |
| **Con.2** | 기본적으로 멤버 함수는 `const`로 만드십시오. |
| **Con.3** | 기본적으로 포인트와 참조는 `const`로 전달하십시오. |
| **Con.4** | 생성 후 값이 변하지 않는다면 `const`를 사용하십시오. |
| **Con.5** | 컴파일 타임에 계산 가능한 값은 `constexpr`을 사용하십시오. |

```cpp
// Con.1 ~ Con.5: 기본 불변성 유지
class Sensor {
public:
    explicit Sensor(std::string id) : id_(std::move(id)) {}

    // Con.2: 멤버 함수는 기본적으로 const
    const std::string& id() const { return id_; }
    double last_reading() const { return reading_; }

    // 수정이 꼭 필요한 경우에만 non-const
    void record(double value) { reading_ = value; }

private:
    const std::string id_;  // Con.4: 생성 후 절대 변하지 않음
    double reading_{0.0};
};

// Con.3: const 참조로 전달
void display(const Sensor& s) {
    std::cout << s.id() << ": " << s.last_reading() << '\n';
}

// Con.5: 컴파일 타임 상수
constexpr double PI = 3.14159265358979;
constexpr int MAX_SENSORS = 256;
```

## 동시성 및 병렬성 (CP.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **CP.2** | 데이터 경합(Data races)을 피하십시오. |
| **CP.3** | 쓰기 가능한 데이터의 명시적 공유를 최소화하십시오. |
| **CP.4** | 스레드가 아닌 작업(Task)의 관점에서 생각하십시오. |
| **CP.8** | 동기화를 위해 `volatile`을 사용하지 마십시오. |
| **CP.20** | RAII를 사용하고, 생으로 `lock()`/`unlock()`을 부르지 마십시오. |
| **CP.21** | 여러 뮤텍스를 획득할 때는 `std::scoped_lock`을 사용하십시오. |
| **CP.22** | 락(Lock)을 소유한 상태에서 정체를 알 수 없는 코드를 호출하지 마십시오. |
| **CP.42** | 조건(Condition) 없이 대기하지 마십시오. |
| **CP.44** | `lock_guard`나 `unique_lock`은 반드시 이름을 지정하십시오. |
| **CP.100** | 절대적으로 필요한 경우가 아니면 락-프리(Lock-free) 프로그래밍을 피하십시오. |

### 안전한 잠금(Locking)

```cpp
// CP.20 + CP.44: 명명된 RAII 락 활용
class ThreadSafeQueue {
public:
    void push(int value) {
        std::lock_guard<std::mutex> lock(mutex_);  // CP.44: 이름 필수!
        queue_.push(value);
        cv_.notify_one();
    }

    int pop() {
        std::unique_lock<std::mutex> lock(mutex_);
        // CP.42: 항상 조건과 함께 대기
        cv_.wait(lock, [this] { return !queue_.empty(); });
        const int value = queue_.front();
        queue_.pop();
        return value;
    }

private:
    std::mutex mutex_;             // CP.50: 뮤텍스는 보호할 데이터와 함께 배치
    std::condition_variable cv_;
    std::queue<int> queue_;
};
```

### 다중 뮤텍스

```cpp
// CP.21: 여러 뮤텍스 획득 시 std::scoped_lock 사용 (데드락 방지)
void transfer(Account& from, Account& to, double amount) {
    std::scoped_lock lock(from.mutex_, to.mutex_);
    from.balance_ -= amount;
    to.balance_ += amount;
}
```

### 안티 패턴
* 동기화 수단으로 `volatile` 사용 (CP.8 -- 하드웨어 I/O 전용임)
* 스레드 분리(Detach) (CP.26 -- 수명 관리가 거의 불가능해짐)
* 이름 없는 락 임시 객체 사용: `std::lock_guard<std::mutex>(m);`는 즉시 소멸됨 (CP.44)
* 콜백 호출 시 락 유지 (CP.22 -- 데드락 위험)
* 충분한 전문가적 식견 없이 락-프리 프로그래밍 시도 (CP.100)

## 템플릿 및 제네릭 프로그래밍 (T.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **T.1** | 추상화 수준을 높이기 위해 템플릿을 사용하십시오. |
| **T.2** | 여러 파라미터 타입에 적용되는 알고리즘 표현에 템플릿을 사용하십시오. |
| **T.10** | 모든 템플릿 파라미터에 '컨셉(Concepts)'을 지정하십시오. |
| **T.11** | 가급적 표준 컨셉을 활용하십시오. |
| **T.13** | 간단한 컨셉에는 속성 구문(Shorthand notation)을 선호하십시오. |
| **T.43** | `typedef`보다 `using`을 선호하십시오. |
| **T.120** | 꼭 필요한 경우에만 템플릿 메타프로그래밍(TMP)을 사용하십시오. |
| **T.144** | 함수 템플릿을 특수화하지 마십시오(대신 오버로딩 사용). |

### 컨셉 (Concepts, C++20)

```cpp
#include <concepts>

// T.10 + T.11: 표준 컨셉으로 템플릿 제약 조건 설정
template<std::integral T>
T gcd(T a, T b) {
    while (b != 0) {
        a = std::exchange(b, a % b);
    }
    return a;
}

// T.13: 속성 컨셉 구문
void sort(std::ranges::random_access_range auto& range) {
    std::ranges::sort(range);
}

// 도메인 전용 제약 조건을 위한 커스텀 컨셉
template<typename T>
concept Serializable = requires(const T& t) {
    { t.serialize() } -> std::convertible_to<std::string>;
};

template<Serializable T>
void save(const T& obj, const std::string& path);
```

### 안티 패턴
* 명시적이지 않은(Unconstrained) 템플릿 사용 (T.47)
* 오버로딩 대신 함수 템플릿 특수화 (T.144)
* `constexpr`로 충분한데 템플릿 메타프로그래밍 사용 (T.120)
* `using` 대신 `typedef` 사용 (T.43)

## 표준 라이브러리 (SL.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **SL.1** | 가능한 한 라이브러리를 활용하십시오. |
| **SL.2** | 타사 라이브러리보다 표준 라이브러리를 우선하십시오. |
| **SL.con.1** | C 스타일 배열보다 `std::array`나 `std::vector`를 선호하십시오. |
| **SL.con.2** | 컨테이너 선택 시 기본적으로 `std::vector`를 우선하십시오. |
| **SL.str.1** | 문자 시퀀스 소유 시 `std::string`을 사용하십시오. |
| **SL.str.2** | 문자 시퀀스 참조 시 `std::string_view`를 사용하십시오. |
| **SL.io.50** | `endl` 대신 `'\n'`을 사용하십시오 (`endl`은 강제 플러시를 수반함). |

```cpp
// SL.con.1 + SL.con.2: C 배열보다 vector/array 선호
const std::array<int, 4> fixed_data{1, 2, 3, 4};
std::vector<std::string> dynamic_data;

// SL.str.1 + SL.str.2: string은 소유, string_view는 관찰
std::string build_greeting(std::string_view name) {
    return "안녕하세요, " + std::string(name) + "님!";
}

// SL.io.50: endl 대신 '\n' 사용
std::cout << "결과: " << value << '\n';
```

## 열거형 (Enum.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **Enum.1** | 매크로보다 열거형(Enum)을 선호하십시오. |
| **Enum.3** | 일반 `enum`보다 `enum class`를 선호하십시오. |
| **Enum.5** | 열거형 항목에 대문자 전용 명칭(ALL_CAPS)을 사용하지 마십시오. |
| **Enum.6** | 이름 없는(Unnamed) 열거형을 피하십시오. |

```cpp
// Enum.3 + Enum.5: 스코프 있는 열거형, 대문자 지양
enum class Color { red, green, blue };
enum class LogLevel { debug, info, warning, error };

// 나쁨: 이름 누출, 매크로와의 충돌 위험
enum { RED, GREEN, BLUE };           // Enum.3, 5, 6 위반
#define MAX_SIZE 100                  // Enum.1 위반 -- constexpr 사용 권장
```

## 소스 파일 및 명명 (SF.*, NL.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **SF.1** | 소스 파일은 `.cpp`, 인터페이스 파일은 `.h`를 사용하십시오. |
| **SF.7** | 헤더 파일의 전역 범위에서 `using namespace`를 쓰지 마십시오. |
| **SF.8** | 모든 `.h` 파일은 `#include` 가드(Guard)를 사용해야 합니다. |
| **SF.11** | 헤더 파일은 자기 완비적(Self-contained)이어야 합니다. |
| **NL.5** | 이름에 타입 정보를 넣지 마십시오 (헝가리안 표기법 금지). |
| **NL.8** | 일관된 명명 스타일을 유지하십시오. |
| **NL.9** | 오직 매크로에만 대문자 전용(ALL_CAPS) 명칭을 사용하십시오. |
| **NL.10** | 명명 규칙으로 `underscore_style`을 우선하십시오. |

### 인클루드 가드

```cpp
// SF.8: 인클루드 가드 (또는 #pragma once)
#ifndef PROJECT_MODULE_WIDGET_H
#define PROJECT_MODULE_WIDGET_H

// SF.11: 자기 완비적 -- 이 헤더에 필요한 모든 다른 헤더를 인클루드
#include <string>
#include <vector>

namespace project::module {

class Widget {
public:
    explicit Widget(std::string name);
    const std::string& name() const;

private:
    std::string name_;
};

}  // namespace project::module

#endif  // PROJECT_MODULE_WIDGET_H
```

### 명명 규칙

```cpp
// NL.8 + NL.10: 일관된 underscore_style
namespace my_project {

constexpr int max_buffer_size = 4096;  // NL.9: 매크로가 아니므로 ALL_CAPS 지양

class tcp_connection {                 // 클래스도 underscore_style
public:
    void send_message(std::string_view msg);
    bool is_connected() const;

private:
    std::string host_;                 // 멤버 변수 끝에는 언더스코어(_) 추가
    int port_;
};

}  // namespace my_project
```

### 안티 패턴
* 헤더 파일 전역 범위에서 `using namespace std;` 사용 (SF.7)
* 인클루드 순서에 따라 컴파일 여부가 달라지는 헤더 파일 (SF.10, SF.11)
* `strName`, `iCount` 같은 헝가리안 표기법 (NL.5)
* 매크로 이외의 용도에 ALL_CAPS 사용 (NL.9)

## 성능 (Per.*)

### 주요 규칙

| 규칙 | 요약 |
|------|---------|
| **Per.1** | 이유 없는 최적화를 하지 마십시오. |
| **Per.2** | 조기 최적화(Premature optimization)를 하지 마십시오. |
| **Per.6** | 측정 데이터 없이 성능에 대해 단정 짓지 마십시오. |
| **Per.7** | 최적화가 용이하도록 설계하십시오. |
| **Per.10** | 정적 타입 시스템을 신뢰하고 활용하십시오. |
| **Per.11** | 연산을 런타임에서 컴파일 타임으로 옮기십시오. |
| **Per.19** | 예측 가능한 방식으로 메모리에 접근하십시오. |

### 가이드라인 예시

```cpp
// Per.11: 가능한 경우 컴파일 타임 연산 처리
constexpr auto lookup_table = [] {
    std::array<int, 256> table{};
    for (int i = 0; i < 256; ++i) {
        table[i] = i * i;
    }
    return table;
}();

// Per.19: 캐시 효율을 위해 연속된 메모리 레이아웃 선호
std::vector<Point> points;           // 좋음: 연속적 배치
std::vector<std::unique_ptr<Point>> indirect_points; // 나쁨: 포인터 추적(Pointer chasing) 수반
```

### 안티 패턴
* 프로파일링 데이터 없이 최적화 수행 (Per.1, Per.6)
* 명확한 추상화 대신 '기교 섞인' 저수준 코드 선택 (Per.4, Per.5)
* 데이터 레이아웃과 캐시 동작 무시 (Per.19)

## 빠른 실무 체크리스트 (Quick Reference Checklist)

C++ 작업을 완료하기 전에 확인하십시오:

* [ ] 생 `new`/`delete`가 없는가 — 스마트 포인터나 RAII 사용 여부 (R.11)
* [ ] 객체가 선언 시점에 초기화되는가 (ES.20)
* [ ] 변수가 기본적으로 `const`/`constexpr`인가 (Con.1, ES.25)
* [ ] 멤버 함수가 가능한 한 `const`로 선언되었는가 (Con.2)
* [ ] 일반 `enum` 대신 `enum class`를 사용했는가 (Enum.3)
* [ ] `0`/`NULL` 대신 `nullptr`를 사용했는가 (ES.47)
* [ ] 정보 손실이 있는 산술 변환(Narrowing)이 없는가 (ES.46)
* [ ] C 스타일 타입 캐스팅을 사용하지 않았는가 (ES.48)
* [ ] 파라미터가 하나인 생성자는 `explicit`인가 (C.46)
* [ ] Rule of Zero 또는 Rule of Five가 지켜졌는가 (C.20, C.21)
* [ ] 베이스 클래스 소멸자가 public virtual이거나 protected non-virtual인가 (C.35)
* [ ] 템플릿에 컨셉(Concepts) 제약 조건이 있는가 (T.10)
* [ ] 헤더 파일 전역 범위에 `using namespace`가 없는가 (SF.7)
* [ ] 헤더 파일에 인클루드 가드가 있고 자기 완비적인가 (SF.8, SF.11)
* [ ] 잠금(Lock) 처리에 RAII(`scoped_lock`/`lock_guard`)를 사용했는가 (CP.20)
* [ ] 예외는 커스텀 타입이며, 값으로 던지고 참조로 잡는가 (E.14, E.15)
* [ ] `std::endl` 대신 `'\n'`을 사용했는가 (SL.io.50)
* [ ] 이름 없는 매직 넘버가 없는가 (ES.45)
