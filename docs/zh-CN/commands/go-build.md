---
description: Go 빌드 에러, go vet 경고 및 linter 문제를 단계적으로 수정합니다. go-build-resolver 에이전트를 호출하여 최소한의 정확한 수정을 수행합니다.
---

# Go 빌드 및 복구 (Build & Fix)

이 명령어는 **go-build-resolver** 에이전트를 호출하여 소스 코드의 변경을 최소화하면서 Go 빌드 에러를 점진적으로 수정합니다.

## 주요 기능

1. **진단 실행**: `go build`, `go vet`, `staticcheck` 등을 실행하여 문제를 파악합니다.
2. **에러 분석**: 에러를 파일별로 그룹화하고 심각도에 따라 정렬합니다.
3. **점진적 수정**: 한 번에 하나의 에러만 집중적으로 수정합니다.
4. **수정 사항 검증**: 매번 코드를 수정한 후 빌드를 재실행하여 해결 여부를 확인합니다.
5. **요약 보고**: 수정된 내역과 남은 현안을 일목요연하게 표시합니다.

## 적용 시점

`/go-build` 명령어를 사용하는 경우:

* `go build ./...` 명령이 에러로 인해 실패할 때
* `go vet ./...` 실행 결과로 잠재적 결함이 보고될 때
* `golangci-lint run` 등의 도구에서 경고가 발생할 때
* 모듈 의존성(`go.mod`)이 꼬였을 때
* 다른 팀원의 코드를 가져온 후 빌드가 되지 않을 때

## 실행되는 진단 명령어

```bash
# 기본 빌드 확인
go build ./...

# 정적 분석 (Standard)
go vet ./...

# 확장 린트 처리 (설치된 경우)
staticcheck ./...
golangci-lint run

# 모듈 관련 이슈 점검
go mod verify
go mod tidy -v
```

---

## 실행 예시

````text
사용자: /go-build

에이전트:
# Go 빌드 에러 해결 프로세스

## 초기 진단 결과
```bash
$ go build ./...
internal/service/user.go:25:15: undefined: UserRepository
internal/handler/api.go:42:9: cannot use x (type string) as type int
internal/handler/api.go:58:2: missing return at end of function
```
발견된 에러: 3개

## 수정 1: 정의되지 않은 식별자 해결
- 파일: internal/service/user.go:25
- 에러: undefined: UserRepository
- 조치: 누락된 패키지 가져오기(import) 추가 및 참조 수정

```bash
$ go build ./...
# 남은 에러: 2개
```

## ... (중략) ...

## 최종 검증
```bash
$ go build ./...
# 빌드 성공!

$ go vet ./...
# 이슈 없음
```

## 최종 요약
| 항목 | 수량 |
|--------|-------|
| 수정된 빌드 에러 | 3 |
| 수정된 Vet 경고 | 0 |
| 수정된 파일 수 | 2 |
| 남은 이슈 | 0 |

빌드 상태: ✅ 성공
````

---

## 자주 발생하는 에러 및 해결책

| 에러 메시지 | 일반적인 해결 방법 |
|-------|-------------|
| `undefined: X` | import 추가 또는 오타 수정 |
| `cannot use X as Y` | 타입 변환(Conversion) 또는 대입 로직 수정 |
| `missing return` | 함수 끝에 return 문 추가 |
| `X does not implement Y` | 누락된 메서드 구현 추가 |
| `import cycle` | 패키지 구조 재설계 |
| `declared but not used` | 변수 제거 또는 실제 사용처 지정 |
| `cannot find package` | `go mod tidy` 실행 |

## 수정 전략

1. **빌드 에러 우선 처리**: 먼저 코드가 컴파일이 되어야 합니다.
2. **Vet 경고 차순 처리**: 의심스러운 구조적 결함을 수정합니다.
3. **린트 경고 마지막 처리**: 코딩 스타일 및 최선 관행을 적용합니다.
4. **한 번에 하나씩**: 각 변경 사항을 즉시 검증합니다.
5. **최소한의 변경**: 거창한 리팩토링보다는 문제 해결에만 집중합니다.

**중단 조건**: 동일 에러가 3회 이상 지속되거나, 수정을 통해 에러가 더 늘어나는 경우, 또는 아키텍처 변경이 필요하다고 판단되면 작업을 멈추고 사용자에게 보고합니다.

---

## 관련 정보

* **호출 에이전트**: `agents/go-build-resolver.md`
* **관련 스킬**: `skills/golang-patterns/`
* **함께 쓰면 좋은 명령어**: `/go-test` (빌드 후 테스트), `/go-review` (품질 검토)
