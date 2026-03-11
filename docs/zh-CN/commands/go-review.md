---
description: 관용적 패턴, 동시성 안전성, 에러 처리 및 보안을 포함한 통합적인 Go 코드 리뷰를 수행합니다. go-reviewer 에이전트를 호출합니다.
---

# Go 코드 리뷰 (Code Review)

이 명령어는 **go-reviewer** 에이전트를 호출하여 Go 언어 전용의 심층적인 코드 리뷰를 수행합니다.

## 주요 기능

1. **Go 변경 사항 식별**: `git diff`를 통해 수정된 `.go` 파일을 찾습니다.
2. **정적 분석 실행**: `go vet`, `staticcheck`, `golangci-lint` 등을 가동합니다.
3. **보안 스캔**: SQL 인젝션, 명령 인젝션, 경합 조건(Race Condition) 등을 점검합니다.
4. **동시성 검토**: Goroutine 안전성, 채널(Channel) 사용 방식, 뮤텍스(Mutex) 패턴을 분석합니다.
5. **관용적 Go(Idiomatic Go) 확인**: Go의 전통적인 관습과 최선 관행을 준수하는지 검증합니다.
6. **보고서 생성**: 발견된 이슈를 심각도별로 분류하여 리포트를 작성합니다.

## 적용 시점

`/go-review` 명령어를 사용하는 경우:

* Go 코드를 작성하거나 수정한 직후
* 변경 사항을 커밋하기 전 최종 점검 시
* Go 코드가 포함된 풀 리퀘스트(PR)를 검토할 때
* 새로운 Go 코드베이스를 처음 분석할 때
* Go다운(Idiomatic) 코딩 패턴을 배우고 싶을 때

## 리뷰 카테고리 (심각도)

### 🔴 심각 (필수 수정)
* SQL / 명령 인젝션 취약점
* 동기화되지 않은 공유 자원 접근 (Race Condition)
* 고루틴 누수 (Goroutine Leak)
* 하드코딩된 자격 증명 (API 키 등)
* 안전하지 않은 포인터(Unsafe Pointer) 사용
* 핵심 경로에서 무시된 에러(Error)

### 🟡 높음 (수정 권장)
* 에러 래핑(Error Wrapping) 시 문맥(Context) 누락
* 에러 반환 대신 `panic` 사용
* 컨텍스트(Context) 전파 누락
* 버퍼 없는 채널로 인한 교착 상태(Deadlock) 위험
* 인터페이스 미충족 에러
* 뮤텍스 보호 누락

### 🔵 중간 (고려 사항)
* 비관용적인(Non-idiomatic) 코드 패턴
* 공개된 요소에 Godoc 주석 누락
* 비효율적인 문자열 결합 (Buffer/Builder 미사용)
* 슬라이스(Slice) 사전 할당 누락
* 테이블 기반 테스트(Table-driven tests) 미사용

---

## 실행되는 자동화 도구

```bash
# 기본 정적 분석
go vet ./...

# 고급 체크 (설치된 경우)
staticcheck ./...
golangci-lint run

# 경합 조건 감지
go build -race ./...

# 보인 취약점 점검
govulncheck ./...
```

---

## 실행 예시

````text
사용자: /go-review

에이전트:
# Go 코드 리뷰 결과 보고서

## 리뷰 대상 파일
- internal/handler/user.go (수정됨)
- internal/service/auth.go (수정됨)

## 발견된 주요 이슈

[🔴 심각] 경합 조건 (Race Condition)
파일: internal/service/auth.go:45
내용: 동기화 처리가 되지 않은 상태에서 공유 맵(Map)에 접근하고 있습니다.
```go
var cache = map[string]*Session{}  // 동시 접근 위험!
```
해결책: `sync.RWMutex` 또는 `sync.Map`을 사용하십시오.

[🟡 높음] 에러 문맥(Context) 누락
파일: internal/handler/user.go:28
내용: 반환되는 에러에 구체적인 상황 설명이 부족합니다.
해결책: `fmt.Errorf("get user %s: %w", userID, err)`와 같이 래핑하십시오.

## 최종 요약
* 심각: 1 | 높음: 1 | 중간: 0
권고: ❌ 심각한 이슈가 해결될 때까지 병합(Merge)을 중단하십시오.
````

---

## 관련 정보

* **호출 에이전트**: `agents/go-reviewer.md`
* **관련 스킬**: `skills/golang-patterns/`, `skills/golang-testing/`
* **함께 쓰면 좋은 명령어**: `/go-test` (테스트 실행), `/go-build` (빌드 수정)
