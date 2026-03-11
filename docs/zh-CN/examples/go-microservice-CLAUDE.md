---
description: PostgreSQL, gRPC 및 Docker를 사용하는 Go 마이크로서비스 프로젝트의 CLAUDE.md 설정 예시입니다.
---

# Go 마이크로서비스 프로젝트 가이드 (CLAUDE.md 예시)

이 파일은 Go 언어 기반의 마이크로서비스 프로젝트에서 Claude Code가 준수해야 할 핵심 지침을 담고 있습니다.

## 프로젝트 개요

**기술 스택:** Go 1.22+, PostgreSQL, gRPC + REST (grpc-gateway), Docker, sqlc (타입 안전한 SQL), Wire (의존성 주입).

**아키텍처:** 도메인(Domain), 저장소(Repository), 서비스(Service), 핸들러(Handler) 계층으로 분리된 클린 아키텍처를 지향합니다. 마이크로서비스 간 통신은 gRPC를 기본으로 하며, 외부 클라이언트를 위해 REST 게이트웨이를 제공합니다.

## 핵심 규칙

### 1. Go 코딩 컨벤션
* **정석 준수**: *Effective Go* 및 *Go Code Review Comments* 가이드를 따르십시오.
* **에러 처리**: `errors.New` 또는 `fmt.Errorf`와 `%w`를 사용해 에러를 래핑하십시오. 에러 메시지 문자열 비교는 금지됩니다.
* **초기화**: `init()` 함수 사용을 지양하고 `main()`이나 생성자에서 명시적으로 초기화하십시오.
* **상태 관리**: 전역 가변 상태를 두지 말고 생성자를 통해 의존성을 주입하십시오.
* **컨텍스트**: `context.Context`는 항상 함수의 첫 번째 인자여야 하며 모든 계층으로 전달되어야 합니다.

### 2. 데이터베이스 (SQL & Migrations)
* **sqlc 활용**: `queries/` 폴더의 순수 SQL을 사용하여 sqlc가 타입 안전한 Go 코드를 생성하게 하십시오.
* **마이그레이션**: `migrations/` 폴더 내의 파일로 데이터베이스 변경을 관리하며 직접적인 스키마 수정은 금지됩니다.
* **트랜잭션**: 여러 단계의 작업에는 `pgx.Tx`를 활용한 트랜잭션을 적용하십시오.
* **보안**: 쿼리 작성 시 항상 파라미터($1, $2 등)를 사용하여 인젝션을 방지하십시오.

### 3. 에러 핸들링 전략
* **반환(Return) 우선**: 정말 복구가 불가능한 경우가 아니면 `panic` 대신 `error`를 반환하십시오.
* **문맥 추가**: `fmt.Errorf("user 생성 중 에러: %w", err)`와 같이 에러에 문맥을 추가하십시오.
* **도메인 에러**: 비즈니스 로직 에러는 `domain/errors.go`에 정의하고, 핸들러 계층에서 gRPC 상태 코드로 매핑하십시오.

## 권장 파일 구조

```text
cmd/server/main.go     # 진입점 및 의존성 주입(Wire)
internal/
  domain/              # 비즈니스 타입 및 인터페이스 (에러 정의 포함)
  service/             # 비즈니스 로직 구현 및 테스트
  repository/          # 데이터 액세스 계층 (sqlc 생성 코드 포함)
  handler/             # gRPC 및 REST 핸들러 구현
proto/                 # Protobuf 정의 파일
queries/               # sqlc용 SQL 쿼리 파일
migrations/            # DB 마이그레이션 스크립트
```

## 주요 패턴 예시

### 테이블 구동 테스트 (Table-Driven Tests)
```go
func TestUserService_Create(t *testing.T) {
    tests := []struct {
        name    string
        req     CreateUserRequest
        setup   func(*MockUserRepo)
        wantErr error
    }{
        // 테스트 케이스 정의
    }
    // 루프를 돌며 t.Run 실행
}
```

## 주요 명령어 (Slash Commands)

* `/go-test`: Go 전용 TDD 워크플로우 실행
* `/go-review`: 관용구(Idiomatic Go), 에러 처리, 동시성 중심의 리뷰
* `/go-build`: Go 빌드 에러 자동 수정 시도
* `/security-scan`: 비밀값 노출 및 보안 취약점 점검

**핵심**: Go 언어의 명료함을 유지하기 위해 함수는 50줄 이내로 유지하고, 모든 내보내기(Exported) 대상에는 문서 주석을 작성하십시오. 동시성 제어 시에는 공유 메모리가 아닌 채널(Channel) 통신을 지향하십시오.
