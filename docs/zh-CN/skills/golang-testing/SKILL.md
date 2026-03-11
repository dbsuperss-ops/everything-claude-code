---
name: golang-testing
description: 테이블 기반 테스트, 서브 테스트, 벤치마크, 퍼즈 테스트(Fuzzing) 및 테스트 커버리지를 포함한 Go 테스트 패턴 가이드입니다. TDD 방법론과 관용적인 Go 관습을 따릅니다.
origin: ECC
---

# Go 테스트 패턴

안정적이고 유지보수가 용이한 테스트 작성을 위한 TDD 기반의 종합 Go 테스트 패턴 가이드입니다.

## 적용 시점

* 새로운 Go 함수나 메서드를 작성할 때
* 기존 코드에 테스트 커버리지를 추가할 때
* 성능이 중요한 코드의 벤치마크를 생성할 때
* 입력값 유효성 검사를 위해 퍼즈 테스트(Fuzzing)를 구현할 때
* Go 프로젝트에서 TDD 워크플로우를 따를 때

## Go의 TDD 워크플로우

### Red-Green-Refactor 사이클

* **RED**: 실패하는 테스트를 먼저 작성합니다.
* **GREEN**: 테스트를 통과시키기 위한 최소한의 코드를 작성합니다.
* **REFACTOR**: 테스트 통과 상태를 유지하며 코드를 개선(리팩토링)합니다.
* **REPEAT**: 다음 요구사항에 대해 위 과정을 반복합니다.

### Go TDD 단계별 예시

```go
// 1단계: 함수 정의 (인터페이스/시그니처)
// calculator.go
package calculator
func Add(a, b int) int { panic("미구현") }

// 2단계: 실패하는 테스트 작성 (RED)
// calculator_test.go
func TestAdd(t *testing.T) {
    got := Add(2, 3)
    want := 5
    if got != want {
        t.Errorf("Add(2, 3) = %d; want %d", got, want)
    }
}

// 3단계: 최소한의 코드 구현 (GREEN)
func Add(a, b int) int { return a + b }

// 4단계: 테스트 실행 및 통과 확인
// $ go test
```

## 테이블 기반 테스트 (Table-Driven Tests)

Go에서 권장되는 표준 테스트 패턴입니다. 적은 코드로 다양한 케이스를 검증할 수 있습니다.

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"양수 더하기", 2, 3, 5},
        {"음수 더하기", -1, -2, -3},
        {"0 더하기", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

## 서브 테스트 및 벤치마크

### 테스트 그룹화 (Subtests)
`t.Run`을 사용하여 관련 테스트를 논리적으로 묶고 픽스처(Setup/Cleanup)를 공유할 수 있습니다.

### 병렬 테스트 실행
```go
func TestParallel(t *testing.T) {
    for _, tt := range tests {
        tt := tt // 고루틴 내 변수 캡처 주의
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel() // 개별 서브 테스트를 병렬로 실행
            // ...
        })
    }
}
```

## 테스트 헬퍼 함수 (Helpers)

```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper() // 호출 지점의 라인 번호를 표시하도록 설정

    db, _ := sql.Open("sqlite3", ":memory:")
    t.Cleanup(func() { db.Close() }) // 테스트 완료 후 자동 정리

    return db
}
```

## 모킹 (Mocking)

Go에서는 인터페이스를 활용하여 의존성을 분리하고 모의 객체를 주입합니다.

```go
type UserRepository interface {
    GetUser(id string) (*User, error)
}

// 테스트용 모킹 구조체
type MockUserRepository struct {
    GetUserFunc func(id string) (*User, error)
}

func (m *MockUserRepository) GetUser(id string) (*User, error) {
    return m.GetUserFunc(id)
}
```

## 벤치마크 (Benchmarks)

함수의 성능을 배정밀도(Nanoseconds per operation)로 측정합니다.

```go
func BenchmarkProcess(b *testing.B) {
    data := generateTestData()
    b.ResetTimer() // 셋업 시간 제외

    for i := 0; i < b.N; i++ {
        Process(data)
    }
}
// 실행: go test -bench=. -benchmem
```

## 퍼즈 테스트 (Fuzzing - Go 1.18+)

무작위 입력값을 생성하여 예기치 못한 에러나 보안 취약점을 발견합니다.

```go
func FuzzParseJSON(f *testing.F) {
    f.Add(`{"name": "test"}`) // 시드(Seed) 데이터 추가
    f.Fuzz(func(t *testing.T, input string) {
        var result map[string]interface{}
        json.Unmarshal([]byte(input), &result)
        // 충돌이나 비정상 종료가 없는지 확인
    })
}
// 실행: go test -fuzz=FuzzParseJSON
```

## 테스트 커버리지

```bash
# 기본 커버리지 확인
go test -cover ./...

# HTML 보고서 생성
go test -coverprofile=coverage.out
go tool cover -html=coverage.out
```

**권장 목표:**
* 비즈니스 핵심 로직: 100%
* 공용 API: 90% 이상
* 전체 평균: 80% 이상

## HTTP 핸들러 테스트

`net/http/httptest` 패키지를 사용하여 실제 서버 가동 없이 핸들러를 테스트합니다.

```go
func TestHealthHandler(t *testing.T) {
    req := httptest.NewRequest("GET", "/health", nil)
    w := httptest.NewRecorder()

    HealthHandler(w, req)

    if w.Code != http.StatusOK {
        t.Errorf("상태 코드 에러: %d", w.Code)
    }
}
```

## 테스트 주요 명령어 요약

| 명령어 | 용도 |
|-------|---------|
| `go test ./...` | 전체 프로젝트 테스트 실행 |
| `go test -v` | 상세 로그 출력 |
| `go test -race` | 데이터 레이스(Race condition) 탐지 |
| `go test -short` | 시간이 오래 걸리는 테스트 제외 |
| `go test -update` | 골든 파일(Golden files) 업데이트 (커스텀 플래그) |
| `go test -count=10` | 테스트 10번 반복 (불안정한 테스트 탐지) |

## 베스트 프랙티스

### 권장 사항 (DO)
* **테스트를 먼저 작성**: TDD 생활화.
* **테이블 기반 테스트 활용**: 코드 중복 최소화.
* **구현이 아닌 동작 테스트**: 내부 코드가 아닌 결과값 검증.
* **`t.Cleanup()` 사용**: 리소스 정리 자동화.
* **명확한 테스트 이름**: `TestUser_Add_DuplicateEmail` 등 의도 명시.

### 금지 사항 (DON'T)
* **비공개(Private) 함수 직접 테스트**: 가급적 공개 API를 통해 검증하십시오.
* **`time.Sleep()` 사용**: 고정 대기 대신 채널이나 상태 확인을 사용하십시오.
* **불안정한(Flaky) 테스트 방치**: 원인을 찾거나 제거하십시오.
* **모든 것을 모킹**: 지나친 모킹은 실제 동작과 괴리가 생깁니다. 필요시 통합 테스트를 병행하십시오.

**기억하십시오**: 테스트는 그 자체로 **살아있는 문서**입니다. 코드가 어떻게 사용되어야 하는지 보여주도록 명확하게 작성하고 주기적으로 업데이트하십시오.
