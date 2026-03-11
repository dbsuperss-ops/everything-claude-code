---
이름: go-build
설명: Go 빌드 에러, go vet 경고, 린터 이슈를 한 번에 하나씩 수정합니다. 최소한의 수술적 수정을 위해 go-build-resolver 에이전트를 호출합니다.
---

# Go 빌드 및 수정 (Go Build and Fix)

이 명령어는 **go-build-resolver** 에이전트를 호출하여 최소한의 변경으로 Go 빌드 에러를 점진적으로 수정합니다.

## 주요 기능

1. **진단 실행**: `go build`, `go vet`, `staticcheck` 실행
2. **에러 분석**: 파일별 그룹화 및 심각도순 정렬
3. **점진적 수정**: 한 번에 하나의 에러씩 수정
4. **결과 검증**: 변경 시마다 빌드 재실행
5. **요약 보고**: 수정된 사항과 남은 사항 표시

## 사용 시점

다음과 같은 경우 `/go-build`를 사용하십시오:
- `go build ./...` 실행 결과 에러가 발생할 때
- `go vet ./...`에서 이슈를 보고할 때
- `golangci-lint run`에서 경고가 표시될 때
- 모듈 의존성이 깨졌을 때
- 빌드를 깨뜨리는 변경 사항을 가져온(pull) 직후

## 실행되는 진단 명령어

```bash
# 기본 빌드 확인
go build ./...

# 정적 분석
go vet ./...

# 확장 린팅 (사용 가능한 경우)
staticcheck ./...
golangci-lint run

# 모듈 이슈 확인
go mod verify
go mod tidy -v
```

## 세션 예시

```text
사용자: /go-build

에이전트:
# Go 빌드 에러 해결

## 초기 진단 결과

```bash
$ go build ./...
internal/service/user.go:25:15: undefined: UserRepository
internal/handler/api.go:42:9: cannot use x (type string) as type int
internal/handler/api.go:58:2: missing return at end of function
```

총 3개의 에러 발견됨.

## 수정 1: 정의되지 않은 식별자 (Undefined Identifier)

파일: internal/service/user.go:25
에러: undefined: UserRepository
원인: 임포트 누락

```go
// 임포트 추가
import "project/internal/repository"

// 변경 전
var repo UserRepository
// 변경 후
var repo repository.UserRepository
```

```bash
$ go build ./...
# 2개의 에러 남음
```

## 수정 2: 타입 불일치 (Type Mismatch)

파일: internal/handler/api.go:42
에러: cannot use x (type string) as type int

```go
// 변경 전
count := params.Get("count")
// 변경 후
countStr := params.Get("count")
count, _ := strconv.Atoi(countStr)
```

```bash
$ go build ./...
# 1개의 에러 남음
```

## 수정 3: 반환값 누락 (Missing Return)

파일: internal/handler/api.go:58
에러: missing return at end of function

```go
func GetUser(id string) (*User, error) {
    if id == "" {
        return nil, ErrInvalidID
    }
    user := findUser(id)
    // 누락된 return 추가
    return user, nil
}
```

```bash
$ go build ./...
# 빌드 성공!
```

## 최종 검증

```bash
$ go vet ./...
# 이슈 없음

$ go test ./...
ok      project/internal/service   0.015s
ok      project/internal/handler   0.023s
```

## 요약

| 지표 | 개수 |
|--------|-------|
| 수정된 빌드 에러 | 3 |
| 수정된 Vet 경고 | 0 |
| 수정된 파일 수 | 2 |
| 남은 이슈 | 0 |

빌드 상태: ✅ 성공(SUCCESS)
```

## 자주 발생하는 에러 및 수정 방법

| 에러 | 일반적인 수정 방법 |
|-------|-------------|
| `undefined: X` | 임포트 추가 또는 오타 수정 |
| `cannot use X as Y` | 타입 변환 또는 할당 수정 |
| `missing return` | return 구문 추가 |
| `X does not implement Y` | 누락된 메서드 구현 |
| `import cycle` | 패키지 구조 재조정 |
| `declared but not used` | 변수 제거 또는 사용 |
| `cannot find package` | `go get` 또는 `go mod tidy` 실행 |

## 수정 전략

1. **빌드 에러 우선** - 코드가 반드시 컴파일되어야 함
2. **Vet 경고 우선순위** - 의심스러운 구조 수정
3. **린트 경고 후순위** - 스타일 및 최선 관행 준수
4. **한 번에 한 가지씩** - 변경 후 반드시 검증
5. **최소한의 변경** - 리팩토링이 아닌 수정에 집중

## 중단 조건

다음과 같은 경우 에이전트는 작업을 중단하고 보고합니다:
- 3회 시도 후에도 동일한 에러가 지속될 때
- 수정 사항이 더 많은 에러를 유발할 때
- 아키텍처 변경이 필요할 때
- 외부 의존성이 누락되었을 때

## 관련 명령어

- `/go-test` - 빌드 성공 후 테스트 실행
- `/go-review` - 코드 품질 리뷰
- `/verify` - 전체 검증 루프 실행

## 참고 문서

- 에이전트: `agents/go-build-resolver.md`
- 스킬: `skills/golang-patterns/`
