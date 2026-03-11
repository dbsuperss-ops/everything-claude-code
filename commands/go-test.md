---
이름: go-test
설명: Go를 위한 TDD 워크플로우를 강제합니다. 테이블 주도 테스트(Table-driven tests)를 먼저 작성한 후 구현합니다. go test -cover를 통해 80% 이상의 커버리지를 확인합니다.
---

# Go TDD 명령어

이 명령어는 관용적인 Go 테스트 패턴을 사용하여 Go 코드에 대한 테스트 주도 개발(TDD) 방법론을 강제합니다.

## 주요 기능

1. **타입/인터페이스 정의**: 함수 시그니처를 먼저 골격만 작성합니다.
2. **테이블 주도 테스트 작성**: 종합적인 테스트 케이스를 생성합니다 (RED 단계).
3. **테스트 실행**: 테스트가 의도한 사유로 실패하는지 확인합니다.
4. **코드 구현**: 테스트를 통과하기 위한 최소한의 코드를 작성합니다 (GREEN 단계).
5. **리팩토링**: 테스트 통과 상태를 유지하며 코드를 개선합니다.
6. **커버리지 확인**: 80% 이상의 커버리지를 확보했는지 확인합니다.

## 사용 시점

다음과 같은 경우 `/go-test`를 사용하십시오:
- 새로운 Go 함수를 구현할 때
- 기존 코드에 테스트 커버리지를 추가할 때
- 버그를 수정할 때 (실패하는 테스트를 먼저 작성)
- 중요한 비즈니스 로직을 구축할 때
- Go에서 TDD 워크플로우를 학습하고 싶을 때

## TDD 사이클

```
RED     → 실패하는 테이블 주도 테스트 작성
GREEN   → 테스트 통과를 위한 최소한의 코드 구현
REFACTOR → 코드를 개선하되 테스트 통과 상태 유지
REPEAT  → 다음 테스트 케이스로 반복
```

## 세션 예시

````
사용자: /go-test 이메일 주소를 검증하는 함수가 필요해

에이전트:
# TDD 세션: 이메일 검증기

## 1단계: 인터페이스 정의

```go
// validator/email.go
package validator

// ValidateEmail은 주어진 문자열이 유효한 이메일 주소인지 확인합니다.
// 유효하면 nil을, 그렇지 않으면 이슈를 설명하는 에러를 반환합니다.
func ValidateEmail(email string) error {
    panic("not implemented")
}
```

## 2단계: 테이블 주도 테스트 작성 (RED)

```go
// validator/email_test.go
package validator

import (
    "testing"
)

func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        // 유효한 이메일
        {"단순 이메일", "user@example.com", false},
        {"서브도메인 포함", "user@mail.example.com", false},
        {"플러스 기호 포함", "user+tag@example.com", false},
        {"점 포함", "first.last@example.com", false},

        // 유효하지 않은 이메일
        {"빈 문자열", "", true},
        {"@ 기호 없음", "userexample.com", true},
        {"도메인 없음", "user@", true},
        {"로컬 파트 없음", "@example.com", true},
        {"@ 기호 두 개", "user@@example.com", true},
        {"공백 포함", "user @example.com", true},
        {"TLD 없음", "user@example", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateEmail(tt.email)
            if tt.wantErr && err == nil {
                t.Errorf("ValidateEmail(%q) = nil; want error", tt.email)
            }
            if !tt.wantErr && err != nil {
                t.Errorf("ValidateEmail(%q) = %v; want nil", tt.email, err)
            }
        })
    }
}
```

## 3단계: 테스트 실행 - 실패 확인

```bash
$ go test ./validator/...

--- FAIL: TestValidateEmail (0.00s)
    --- FAIL: TestValidateEmail/단순_이메일 (0.00s)
        panic: not implemented

FAIL
```

✓ 예상대로 테스트가 실패(panic)했습니다.

## 4단계: 최소한의 코드 구현 (GREEN)

```go
// validator/email.go
package validator

import (
    "errors"
    "regexp"
)

var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)

var (
    ErrEmailEmpty   = errors.New("이메일은 비어 있을 수 없습니다")
    ErrEmailInvalid = errors.New("이메일 형식이 유효하지 않습니다")
)

func ValidateEmail(email string) error {
    if email == "" {
        return ErrEmailEmpty
    }
    if !emailRegex.MatchString(email) {
        return ErrEmailInvalid
    }
    return nil
}
```

## 5단계: 테스트 실행 - 성공 확인

```bash
$ go test ./validator/...

PASS
ok      project/validator    0.003s
```

✓ 모든 테스트를 통과했습니다!

## 6단계: 커버리지 확인

```bash
$ go test -cover ./validator/...

PASS
coverage: 100.0% of statements
ok      project/validator    0.003s
```

✓ 커버리지: 100%

## TDD 완료!
````

## 테스트 패턴

### 테이블 주도 테스트 (Table-Driven Tests)
```go
tests := []struct {
    name     string
    input    InputType
    want     OutputType
    wantErr  bool
}{
    {"케이스 1", input1, want1, false},
    {"케이스 2", input2, want2, true},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Function(tt.input)
        // 단언문(Assertions) 수행
    })
}
```

### 병렬 테스트 (Parallel Tests)
```go
for _, tt := range tests {
    tt := tt // 캡처
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // 테스트 본문
    })
}
```

### 테스트 헬퍼 (Test Helpers)
```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db := createDB()
    t.Cleanup(func() { db.Close() })
    return db
}
```

## 커버리지 관련 명령어

```bash
# 기본 커버리지 확인
go test -cover ./...

# 커버리지 프로필 생성
go test -coverprofile=coverage.out ./...

# 브라우저에서 결과 보기
go tool cover -html=coverage.out

# 함수별 커버리지 확인
go tool cover -func=coverage.out

# 경쟁 상태 탐지 포함
go test -race -cover ./...
```

## 커버리지 목표 수치

| 코드 유형 | 목표 수치 |
|-----------|--------|
| 핵심 비즈니스 로직 | 100% |
| 퍼블릭 API | 90% 이상 |
| 일반 코드 | 80% 이상 |
| 자동 생성된 코드 | 제외 |

## TDD 최선 관행

**권장 사항 (DO):**
- 구현 전 반드시 테스트를 먼저 작성하십시오.
- 변경 사항마다 테스트를 실행하십시오.
- 종합적인 커버리지를 위해 테이블 주도 테스트를 사용하십시오.
- 구현 상세가 아닌 동작(Behavior)을 테스트하십시오.
- 엣지 케이스(empty, nil, 최대값 등)를 포함하십시오.

**지양 사항 (DON'T):**
- 테스트보다 구현을 먼저 하지 마십시오.
- RED 단계를 건너뛰지 마십시오.
- 프라이빗 함수를 직접 테스트하지 마십시오.
- 테스트 내에서 `time.Sleep`을 사용하지 마십시오.
- 불안정한 테스트를 방치하지 마십시오.

## 관련 명령어

- `/go-build` - 빌드 에러 수정
- `/go-review` - 구현 후 코드 리뷰
- `/verify` - 전체 검증 루프 실행

## 관련 문서

- 스킬: `skills/golang-testing/`
- 스킬: `skills/tdd-workflow/`
