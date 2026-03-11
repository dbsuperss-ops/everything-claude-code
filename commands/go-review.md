---
이름: go-review
설명: 관용적인 패턴, 동시성 안전성, 에러 처리 및 보안에 대해 종합적인 Go 코드 리뷰를 수행합니다. go-reviewer 에이전트를 호출합니다.
---

# Go 코드 리뷰 (Go Code Review)

이 명령어는 종합적인 Go 전용 코드 리뷰를 위해 **go-reviewer** 에이전트를 호출합니다.

## 주요 기능

1. **Go 변경 사항 식별**: `git diff`를 통해 수정된 `.go` 파일을 찾습니다.
2. **정적 분석 실행**: `go vet`, `staticcheck`, `golangci-lint` 등을 실행합니다.
3. **보안 스캔**: SQL 인젝션, 명령 인젝션, 경쟁 상태(Race condition) 등을 점검합니다.
4. **동시성 리뷰**: 고루틴 안전성, 채널 사용법, 뮤텍스(Mutex) 패턴을 분석합니다.
5. **관용적 Go(Idiomatic Go) 확인**: 코드가 Go 컨벤션 및 최선 관행을 따르는지 확인합니다.
6. **보고서 생성**: 이슈를 심각도별로 분류하여 보고합니다.

## 사용 시점

다음과 같은 경우 `/go-review`를 사용하십시오:
- Go 코드를 작성하거나 수정했을 때
- Go 변경 사항을 커밋하기 전
- Go 코드가 포함된 Pull Request를 리뷰할 때
- 새로운 Go 코드베이스에 온보딩할 때
- 관용적인 Go 패턴을 학습하고 싶을 때

## 리뷰 카테고리

### 치명적 (CRITICAL - 반드시 수정)
- SQL/명령 인젝션 취약점
- 동기화되지 않은 공유 상태 접근(경쟁 상태)
- 고루틴 누수 (Goroutine leaks)
- 하드코딩된 자격 증명 (API 키 등)
- 안전하지 않은 포인터(unsafe) 사용
- 핵심 경로에서의 에러 무시

### 높음 (HIGH - 수정 권장)
- 컨텍스트 정보가 누락된 에러 반환 (에러 래핑 누락)
- 에러 반환 대신 Panic 사용
- 컨텍스트(Context) 전파 누락
- 데드락을 유발할 수 있는 버퍼 없는 채널 사용
- 인터페이스 미준수 에러
- 뮤텍스 보호 누락

### 보통 (MEDIUM - 고려 사항)
- 관용적이지 않은 코드 패턴
- 내보내기(Export) 대상에 대한 godoc 주석 누락
- 비효율적인 문자열 연결 (루프 내 `+` 등)
- 슬라이스 사전 할당(Preallocate) 누락
- 테이블 주도 테스트(Table-driven tests) 미사용

## 자동 검사 도구

```bash
# 정적 분석
go vet ./...

# 고급 검사 (설치된 경우)
staticcheck ./...
golangci-lint run

# 경쟁 상태 탐지
go build -race ./...

# 보안 취약점 점검
govulncheck ./...
```

## 사용 예시

```text
사용자: /go-review

에이전트:
# Go 코드 리뷰 보고서

## 리뷰된 파일
- internal/handler/user.go (수정됨)
- internal/service/auth.go (수정됨)

## 정적 분석 결과
✓ go vet: 이슈 없음
✓ staticcheck: 이슈 없음

## 이슈 발견 사항

[치명적 - CRITICAL] 경쟁 상태 (Race Condition)
파일: internal/service/auth.go:45
이슈: 동기화 없이 공유 맵(Map)에 접근함
```go
var cache = map[string]*Session{}  // 동시 접근 발생!

func GetSession(id string) *Session {
    return cache[id]  // 경쟁 상태 발생 여지 있음
}
```
수정 제안: sync.RWMutex 또는 sync.Map 사용
```go
var (
    cache   = map[string]*Session{}
    cacheMu sync.RWMutex
)

func GetSession(id string) *Session {
    cacheMu.RLock()
    defer cacheMu.RUnlock()
    return cache[id]
}
```

[높음 - HIGH] 에러 컨텍스트 누락
파일: internal/handler/user.go:28
이슈: 컨텍스트 정보 없이 에러가 반환됨
```go
return err  // 맥락 정보가 없음
```
수정 제안: 에러 정보 래핑
```go
return fmt.Errorf("get user %s: %w", userID, err)
```

## 요약
- 치명적(CRITICAL): 1
- 높음(HIGH): 1
- 보통(MEDIUM): 0

판정: ❌ 치명적인 이슈가 수정될 때까지 머지(Merge)를 차단하십시오.
```

## 승인 기준

| 상태 | 조건 |
|--------|-----------|
| ✅ 승인(Approve) | 치명적(CRITICAL) 또는 높음(HIGH) 이슈 없음 |
| ⚠️ 경고(Warning) | 보통(MEDIUM) 이슈만 발견됨 (주의하여 머지) |
| ❌ 차단(Block) | 치명적(CRITICAL) 또는 높음(HIGH) 이슈 발견됨 |

## 다른 명령어와의 통합

- 테스트 통과 여부 확인을 위해 `/go-test` 먼저 실행
- 빌드 에러 발생 시 `/go-build` 사용
- 커밋 전 반드시 `/go-review` 실행
- Go 전용 사항이 아닌 일반적인 사항은 `/code-review` 활용

## 관련 문서

- 에이전트: `agents/go-reviewer.md`
- 스킬: `skills/golang-patterns/`, `skills/golang-testing/`
