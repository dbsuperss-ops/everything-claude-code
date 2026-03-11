---
description: Axum, SQLx 및 PostgreSQL을 사용하는 Rust 기반 API 서비스 프로젝트의 CLAUDE.md 설정 예시입니다.
---

# Rust API 서비스 프로젝트 가이드 (CLAUDE.md 예시)

이 파일은 Rust 언어 기반의 백엔드 프로젝트에서 Claude Code가 준수해야 할 핵심 지침을 담고 있습니다.

## 프로젝트 개요

**기술 스택:** Rust 1.78+, Axum (Web 프레임워크), SQLx (비동기 DB), PostgreSQL, Tokio (런타임), Docker.

**아키텍처:** 핸들러(Handler), 서비스(Service), 저장소(Repository) 계층으로 분리된 구조를 가집니다. Axum을 HTTP 레이어로 사용하며, SQLx를 통해 컴파일 타임에 쿼리 유효성을 검증합니다.

## 핵심 규칙

### 1. Rust 코딩 컨벤션
* **에러 라이브러리**: 라이브러리 수준의 에러 정의에는 `thiserror`를, 바이너리 실행부나 테스트에서는 `anyhow`를 사용하십시오.
* **안전한 전파**: 프로덕션 코드에서 `.unwrap()`이나 `.expect()` 사용을 금지하고 `?` 연산자로 에러를 전파하십시오.
* **소유권**: 함수 인자로는 `String`보다 `&str`을 선호하며, 소유권 이전이 필요한 경우에만 `String`을 반환하십시오.
* **린트 검사**: `clippy`를 사용하여 모든 경고를 해결하십시오. (`#![deny(clippy::all)]` 권장)
* **Unsafe 금지**: 명확한 사유를 담은 `// SAFETY:` 주석 없이는 `unsafe` 블록을 사용하지 마십시오.

### 2. 데이터베이스 (SQLx)
* **정적 검증**: `sqlx::query!` 또는 `query_as!` 매크로를 사용하여 쿼리의 컴파일 타임 타입 체크를 보장하십시오.
* **마이그레이션**: `migrations/` 폴더 내의 파일로 스키마를 관리하며 `sqlx migrate`를 활용하십시오.
* **연결 관리**: `sqlx::Pool<Postgres>`를 공유 상태로 사용하며 요청마다 새로운 연결을 생성하지 마십시오.
* **보안**: 쿼리 작성 시 항상 `$1`, `$2` 등의 파라미터를 사용하여 SQL 인젝션을 방지하십시오.

### 3. 에러 핸들링 및 로깅
* **도메인 에러**: 모듈별로 `thiserror`를 사용한 에러 열거형(Enum)을 정의하십시오.
* **응답 매핑**: `IntoResponse`를 구현하여 에러를 적절한 HTTP 상태 코드와 메시지로 변환하되 내부 상세 구현은 노출하지 마십시오.
* **구조화된 로깅**: `println!` 대신 `tracing` 크레이트를 사용하여 로그를 남기십시오.

## 권장 파일 구조

```text
src/
  main.rs              # 진입점 및 서버 설정
  router.rs            # Axum 라우터 및 경로 정의
  handlers/            # 비즈니스 로직을 호출하는 얇은 핸들러 계층
  services/            # 핵심 비즈니스 로직 구현
  repositories/        # 데이터베이스 액세스 (SQLx 쿼리)
  domain/              # 도메인 모델 및 에러 정의
migrations/            # DB 마이그레이션 스크립트
tests/                 # 통합 테스트 전용 폴더
```

## 주요 패턴 예시

### 계층별 구현 패턴
```rust
// Repository: 데이터 액세스
pub async fn find_by_email(&self, email: &str) -> Result<Option<User>, sqlx::Error> {
    sqlx::query_as!(User, "SELECT * FROM users WHERE email = $1", email)
}

// Service: 비즈니스 로직
pub async fn create(&self, req: CreateUserRequest) -> Result<User, AppError> {
    // 유효성 검사 및 데이터 가공 로직
}
```

## 주요 명령어 (Slash Commands)

* `/verify`: 빌드, clippy, 테스트 및 보안 취약점 전체 점검
* `/tdd`: cargo test 기반의 테스트 주도 개발 워크플로우
* `/code-review`: Rust 관용구 및 소유권 모델 중심의 코드 리뷰
* `/plan`: 신규 기능 개발(예: 결제 모듈 추가) 계획 수립

**핵심**: Rust의 강력한 타입 시스템과 소유권 개념을 최대한 활용하여 런타임 에러를 방지하고, 모든 동시성 제어는 `Arc`와 `Mutex/RwLock`을 적절히 사용하여 안전하게 처리하십시오.
