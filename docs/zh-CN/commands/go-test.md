---
description: Go 언어를 위한 TDD(테스트 주도 개발) 워크플로우를 강제합니다. 테이블 기반 테스트를 먼저 작성한 후 구현하며, go test -cover를 통해 80% 이상의 커버리지를 보장합니다.
---

# Go TDD 명령어

이 명령어는 Go 언어 특유의 테스트 패턴을 활용하여 테스트 주도 개발(TDD) 방법론을 적용합니다.

## 주요 단계

1. **타입/인터페이스 정의**: 구현할 함수의 시그니처를 먼저 설계합니다.
2. **테이블 기반 테스트 작성**: 다양한 테스트 케이스를 포함하는 통합 테스트 코드를 작성합니다. (**RED** 단계)
3. **테스트 실행**: 구현되지 않았으므로 실패해야 함을 확인합니다. (정상적인 실패 확인)
4. **최소 구현**: 테스트를 통과하기 위한 가장 단순한 코드를 작성합니다. (**GREEN** 단계)
5. **리팩토링**: 테스트 성공 상태를 유지하면서 코드를 개선합니다.
6. **커버리지 확인**: `go test -cover`를 통해 80% 이상의 테스트 커버리지를 달성했는지 확인합니다.

## 적용 시점

`/go-test` 명령어를 사용하는 경우:

* 새로운 Go 함수를 구현할 때
* 기존 코드의 테스트 커버리지를 높여야 할 때
* 버그를 수정할 때 (실패하는 테스트를 먼저 작성)
* 핵심 비즈니스 로직을 구축할 때
* Go에서의 표준적인 TDD 방식을 학습하고 싶을 때

## TDD 사이클

```text
RED     → 실패하는 테이블 기반 테스트 작성
GREEN   → 테스트 통과를 위한 최소한의 코드 구현
REFACTOR → 테스트 통과 상태를 유지하며 코드 품질 개선
REPEAT  → 다음 테스트 케이스로 반복
```

---

## 실행 예시

````text
사용자: /go-test 이메일 주소 유효성 검사 함수가 필요해

에이전트:
# TDD 세션: 이메일 검증기 (Email Validator)

## 1단계: 인터페이스 정의
```go
// validator/email.go
func ValidateEmail(email string) error {
    panic("not implemented")
}
```

## 2단계: 테이블 기반 테스트 작성 (RED)
```go
// validator/email_test.go
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        {"정상 이메일", "user@example.com", false},
        {"잘못된 형식", "userexample.com", true},
        {"빈 문자열", "", true},
    }
    // ... 루프를 돌며 t.Run 실행 ...
}
```

## 3단계: 테스트 실행 - 실패 확인
```bash
$ go test ./validator/...
--- FAIL: TestValidateEmail (0.00s)
    panic: not implemented
```
✓ 의도대로 실패함(Panic).

## 4단계: 최소 구현 (GREEN)
```go
// 정규표현식 등을 활용해 이메일 검증 로직 구현
```

## 5단계: 테스트 실행 - 성공 확인
```bash
$ go test -cover ./validator/...
PASS
coverage: 100.0% of statements
```
✓ 모든 테스트 통과 및 커버리지 달성!
````

---

## 주요 테스트 패턴

### 테이블 기반 테스트 (Table-Driven Tests)
```go
tests := []struct {
    name     string
    input    InputType
    want     OutputType
    wantErr  bool
}{
    {"case 1", input1, want1, false},
    {"case 2", input2, want2, true},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Function(tt.input)
        // 검증 로직
    })
}
```

### 병렬 테스트 (Parallel Tests)
```go
for _, tt := range tests {
    tt := tt // 변수 캡처
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // 테스트 본문
    })
}
```

## 커버리지 관련 명령어

```bash
# 기본 커버리지 확인
go test -cover ./...

# 커버리지 프로필 생성
go test -coverprofile=coverage.out ./...

# 브라우저에서 시각적으로 확인
go tool cover -html=coverage.out

# 함수별 커버리지 요약
go tool cover -func=coverage.out

# 경합 조건 감지 및 커버리지 병행
go test -race -cover ./...
```

## 권장 커버리지 목표

| 코드 성격 | 목표치 |
|-----------|--------|
| 핵심 비즈니스 로직 | 100% |
| 공개용(Public) API | 90% 이상 |
| 일반적인 코드 | 80% 이상 |
| 자동 생성된 코드 | 제외 |

**핵심**: TDD는 코드 작성 전 '어떻게 작동해야 하는가'를 정의하는 과정입니다. 테이블 기반 테스트를 활용하여 엣지 케이스를 빠짐없이 검증하십시오.
