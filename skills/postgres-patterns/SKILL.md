---
name: postgres-patterns
description: PostgreSQL 쿼리 최적화, 스키마 설계, 인덱싱 및 보안을 위한 데이터베이스 패턴 가이드입니다. (Supabase 최선 관행 기준)
origin: ECC
---

# PostgreSQL 패턴 (PostgreSQL Patterns)

PostgreSQL 최선 관행에 대한 빠른 참조 가이드입니다. 상세한 가이드는 `database-reviewer` 에이전트를 활용하십시오.

## 활성화 시점

- SQL 쿼리나 마이그레이션 작성 시
- 데이터베이스 스키마 설계 시
- 느린 쿼리(Slow query) 트러블슈팅 시
- 행 레벨 보안(RLS) 구현 시
- 커넥션 풀링(Connection pooling) 설정 시

## 요약 가이드

### 1. 인덱스 퀵 시트
- `WHERE col = value`: **B-tree** (기본값)
- `WHERE a = x AND b > y`: **복합 인덱스 (Composite)**. 일치 조건 컬럼을 앞에, 범위 조건 컬럼을 뒤에 배치하십시오.
- `WHERE jsonb @> '{}'`: **GIN** 인덱스 사용.
- 시계열 데이터 범위 조회: **BRIN** 인덱스 고려.

### 2. 권장 데이터 타입
- **ID**: `bigint` (권장), 무작위 UUID는 성능 저하 유발 가능.
- **문자열**: `text` (권장), `varchar(255)`는 불필요한 제약입니다.
- **타임스탬프**: 반드시 `timestamptz`를 사용하십시오.
- **금액**: 정확도를 위해 `numeric(10,2)` 등을 사용하십시오. `float`는 지양하십시오.

### 3. 주요 패턴
- **커버링 인덱스(Covering Index)**: `INCLUDE` 절을 사용하여 테이블 조회 없이 인덱스만으로 쿼리 결과를 반환하게 합니다.
- **부분 인덱스(Partial Index)**: `WHERE` 절을 사용하여 특정 조건(예: `deleted_at IS NULL`)을 만족하는 행만 인덱싱하여 크기를 줄입니다.
- **UPSERT**: `ON CONFLICT (...) DO UPDATE` 구문을 사용하여 중복 키 발생 시 업데이트를 수행합니다.
- **커서 페이지네이션**: `OFFSET` 대신 `WHERE id > $last_id`와 `LIMIT`을 사용하여 대량 데이터에서도 $O(1)$ 성능을 유지하십시오.
- **큐 프로세싱**: `FOR UPDATE SKIP LOCKED`를 사용하여 여러 프로세스가 동일한 작업을 중복 처리하지 않도록 합니다.

### 4. 안티 패턴 및 성능 점검
- 인덱스가 없는 외래 키(Foreign keys)를 찾아 인덱싱하십시오.
- `pg_stat_statements`를 통해 실행 시간이 긴 쿼리를 주기적으로 확인하십시오.
- `pg_stat_user_tables`를 통해 테이블 블로트(Bloat) 현상을 모니터링하십시오.

### 5. 설정 템플릿 (보안 및 성능)
- `max_connections`, `work_mem` 등을 시스템 리소스에 맞춰 조정하십시오.
- `idle_in_transaction_session_timeout`과 `statement_timeout`을 설정하여 자원 낭비를 방지하십시오.
- 보안을 위해 `public` 스키마로부터 모든 권한을 회수(`REVOKE ALL`)하고 명시적으로 권한을 부여하십시오.

**기억하십시오**: 데이터베이스 성능의 핵심은 인덱싱과 올바른 데이터 타입 선택에 있습니다. 쿼리 실행 계획(EXPLAIN ANALYZE)을 상시 확인하십시오.
    
