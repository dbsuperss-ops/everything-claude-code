---
name: golang-patterns
description: 견고하고 효율적이며 유지보수가 용이한 Go 애플리케이션 구축을 위한 관용적인(Idiomatic) Go 패턴, 베스트 프랙티스 및 규칙 가이드입니다.
origin: ECC
---

# Go 개발 패턴

견고하고 효율적이며 유지보수가 용이한 애플리케이션을 구축하기 위한 관용적인(Idiomatic) Go 패턴과 베스트 프랙티스입니다.

## 적용 시점

* 새로운 Go 코드를 작성할 때
* Go 코드 리뷰를 수행할 때
* 기존 Go 코드를 리팩토링할 때
* Go 패키지나 모듈을 설계할 때

## 핵심 원칙

### 1. 단순함과 명확함 (Simplicity and Clarity)
Go는 기교보다 단순함을 지향합니다. 코드는 읽기 쉽고 의도가 명확해야 합니다.

```go
// ✅ 권장: 명확하고 직접적인 방식
func GetUser(id string) (*User, error) {
    user, err := db.FindUser(id)
    if err != nil {
        return nil, fmt.Errorf("get user %s: %w", id, err)
    }
    return user, nil
}

// ❌ 나쁨: 불필요하게 복잡한 방식
func GetUser(id string) (*User, error) {
    return func() (*User, error) {
        if u, e := db.FindUser(id); e == nil {
            return u, nil
        } else {
            return nil, e
        }
    }()
}
```

### 2. 제로 값(Zero Value)을 유용하게 만들기
타입을 설계할 때 별도의 초기화 없이도 기본값(Zero value) 상태에서 즉시 사용 가능하도록 만드십시오.

```go
// ✅ 권장: 기본값이 유용함
type Counter struct {
    mu    sync.Mutex
    count int // 기본값 0, 즉시 사용 가능
}

func (c *Counter) Inc() {
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}

// sync.Mutex나 bytes.Buffer 등은 선언만으로 즉시 사용 가능하도록 설계됨
var buf bytes.Buffer
buf.WriteString("hello")
```

### 3. 인터페이스로 받고 구조체로 반환하기 (Accept interface, Return struct)
함수는 인터페이스를 매개변수로 받고, 구체적인 타입(구조체 포인터 등)을 반환하는 것이 좋습니다.

```go
// ✅ 권장: 인터페이스를 받고 구체 타입을 반환
func ProcessData(r io.Reader) (*Result, error) {
    data, err := io.ReadAll(r)
    if err != nil {
        return nil, err
    }
    return &Result{Data: data}, nil
}
```

## 에러 처리 패턴

### 컨텍스트를 포함한 에러 래핑 (Error Wrapping)
`fmt.Errorf`와 `%w`를 사용하여 에러 발생 지점의 문맥을 함께 전달하십시오.

```go
func LoadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("config 파일 로드 실패 (%s): %w", path, err)
    }
    // ...
}
```

### errors.Is 및 errors.As 활용
에러의 원인을 확인하거나 특정 에러 타입으로 변환할 때 사용합니다.

```go
if errors.Is(err, sql.ErrNoRows) {
    // 결과 없음 처리
}

var valErr *ValidationError
if errors.As(err, &valErr) {
    // 유효성 검사 에러 상세 처리
}
```

## 동시성 패턴 (Concurrency)

### 워커 풀 (Worker Pool)
```go
func WorkerPool(jobs <-chan Job, results chan<- Result, numWorkers int) {
    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                results <- process(job)
            }
        }()
    }
    wg.Wait()
    close(results)
}
```

### 취소 및 타임아웃을 위한 Context 활용
```go
func FetchWithTimeout(ctx context.Context, url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    // ctx를 지원하는 네트워크 요청 실행
}
```

### 고루틴 누수(Leak) 방지
컨텍스트가 취소되었을 때 고루틴이 영원히 차단되지 않도록 `select` 문을 적절히 사용하십시오.

## 인터페이스 설계

* **작고 집중된 인터페이스**: `io.Reader`, `io.Writer`처럼 메서드 1-2개짜리 인터페이스가 강력합니다.
* **사용자 쪽에서 인터페이스 정의**: 패키지 제공자가 아닌, 해당 기능을 사용하는 곳(Consumer)에서 필요한 메서드만 추려 인터페이스를 정의하십시오. (Duck Typing 활용)

## 패키지 조직 및 명명 규칙

* **표준 레이아웃**: `cmd/`(실행 파일), `internal/`(비공개 로직), `pkg/`(공개 라이브러리) 구분을 따르십시오.
* **패키지 이름**: 짧고 소문자이며 언더바(`_`)가 없는 이름을 사용하십시오 (예: `http`, `json`, `user`).
* **패키지 레벨 상태 지양**: 전역 변수나 `init()` 함수를 통한 상태 공유보다는 의존성 주입(Dependency Injection)을 선호하십시오.

## 메모리 및 성능 최적화

* **슬라이스 미리 할당**: 크기를 알 수 있다면 `make([]T, 0, capacity)`로 미리 메모리를 확보하여 재할당 오버헤드를 줄이십시오.
* **sync.Pool 활용**: 빈번하게 생성되고 해제되는 객체(예: 버퍼)는 풀에 담아 재사용하십시오.
* **strings.Builder 사용**: 루프 안에서 문자열을 합칠 때는 `+` 연산자 대신 `strings.Builder`를 사용하십시오.

## 빠른 참조: Go의 관용구(Idioms)

| 관용구 | 설명 |
|-------|-------------|
| **Accept Interface, Return Struct** | 입력은 유연하게(인터페이스), 출력은 명확하게(구조체) |
| **Error as Value** | 에러를 예외(Exception)가 아닌 일반적인 값으로 취급 |
| **Share memory by communicating** | 고루틴 간 협력 시 채널을 사용 |
| **Effective Zero Value** | 초기화 없이도 동작하는 타입 설계 |
| **Early Return** | 에러를 먼저 처리하여 코드 깊이(Indentation) 축소 |
| **Gofmt is Friend** | 항상 `gofmt` 또는 `goimports`로 포맷을 통일 |

## 피해야 할 안티 패턴

* **Naked Returns**: 긴 함수에서 반환값 이름을 명시하지 않는 `return`은 가독성을 해칩니다.
* **제어 흐름으로 Panic 사용**: 복구 가능한 상황에서는 항상 `error`를 반환하십시오.
* **구조체 내부에 Context 포함**: 컨텍스트는 항상 함수의 첫 번째 매개변수로 전달되어야 합니다.
* **포인터와 값 리시버 혼용**: 한 타입에 대해서는 일관성 있게 하나만 선택하십시오.

**기억하십시오**: Go 코드는 "지루할 정도로" 예측 가능하고 일관되며 이해하기 쉬워야 합니다. 의구심이 들 때는 단순한 길을 선택하십시오.
