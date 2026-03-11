---
name: postgres-patterns
description: 쿼리 최적화, 스키마 설계, 인덱싱 및 보안을 위한 PostgreSQL 데이터베이스 패턴 가이드입니다. Supabase의 베스트 프랙티스를 기반으로 합니다.
origin: ECC
---

# PostgreSQL 패턴

PostgreSQL 베스트 프랙티스 빠른 참조 가이드입니다. 상세한 가이드가 필요한 경우 `database-reviewer` 에이전트를 사용하십시오.

## 적용 시점

* SQL 쿼리 작성 또는 마이그레이션 도구를 사용할 때
* 데이터베이스 스키마를 설계할 때
* 느린 쿼리(Slow query)를 문제 해결(Troubleshoot) 할 때
* 행 수준 보안(RLS, Role Level Security) 정책을 구현할 때
* 커넥션 풀(Connection pool) 설정을 튜닝할 때

## 빠른 참조 (Quick Reference)

### 인덱스 치트 시트 (Index Cheat Sheet)

| 쿼리 패턴 | 인덱스 타입 | 예시 |
|--------------|------------|---------|
| `WHERE col = value` | B-tree (기본값) | `CREATE INDEX idx ON t (col)` |
| `WHERE col > value` | B-tree | `CREATE INDEX idx ON t (col)` |
| `WHERE a = x AND b > y` | 복합 인덱스 | `CREATE INDEX idx ON t (a, b)` |
| `WHERE jsonb @> '{}'` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| `WHERE tsv @@ query` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| 시계열 범위 조회 | BRIN | `CREATE INDEX idx ON t USING brin (col)` |

### 데이터 타입 가이드

| 사용 사례 | 권장 타입 | 피해야 할 타입 |
|----------|-------------|-------|
| 식별자 (ID) | `bigint` | `int`, 랜덤 UUID |
| 문자열 | `text` | `varchar(255)` |
| 타임스탬프 | `timestamptz` | `timestamp` |
| 통화/금액 | `numeric(10,2)` | `float` |
| 플래그(여부) | `boolean` | `varchar`, `int` |

### 주요 패턴

**복합 인덱스 순서 (Composite Index Order):**
```sql
-- 일치(Equality) 컬럼을 먼저 두고, 범위(Range) 컬럼을 뒤에 둡니다.
CREATE INDEX idx ON orders (status, created_at);
-- 효과적인 경우: WHERE status = 'pending' AND created_at > '2024-01-01'
```

**커버링 인덱스 (Covering Index):**
```sql
CREATE INDEX idx ON users (email) INCLUDE (name, created_at);
-- SELECT email, name, created_at 조회 시 테이블 히트 없이 인덱스만으로 처리 가능
```

**부분 인덱스 (Partial Index):**
```sql
CREATE INDEX idx ON users (email) WHERE deleted_at IS NULL;
-- 인덱스 크기를 줄이고, 활성 사용자만 포함하여 성능 향상
```

**RLS 정책 (최적화 버전):**
```sql
CREATE POLICY policy ON orders
  USING ((SELECT auth.uid()) = user_id);  -- SELECT로 감싸서 성능 최적화!
```

**UPSERT (Insert or Update):**
```sql
INSERT INTO settings (user_id, key, value)
VALUES (123, 'theme', 'dark')
ON CONFLICT (user_id, key)
DO UPDATE SET value = EXCLUDED.value;
```

**커서 기반 페이지네이션:**
```sql
SELECT * FROM products WHERE id > $last_id ORDER BY id LIMIT 20;
-- OFFSET 방식(O(n))보다 효율적인 O(1) 성능
```

**큐(Queue) 처리:**
```sql
UPDATE jobs SET status = 'processing'
WHERE id = (
  SELECT id FROM jobs WHERE status = 'pending'
  ORDER BY created_at LIMIT 1
  FOR UPDATE SKIP LOCKED
) RETURNING *;
```

### 안티 패턴 감지 (Monitoring)

```sql
-- 인덱스가 없는 외래 키 찾기
SELECT conrelid::regclass, a.attname
FROM pg_constraint c
JOIN pg_attribute a ON a.attrelid = c.conrelid AND a.attnum = ANY(c.conkey)
WHERE c.contype = 'f'
  AND NOT EXISTS (
    SELECT 1 FROM pg_index i
    WHERE i.indrelid = c.conrelid AND a.attnum = ANY(i.indkey)
  );

-- 느린 쿼리 통계 확인
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
WHERE mean_exec_time > 100
ORDER BY mean_exec_time DESC;

-- 테이블 데드 튜플(Bloat) 확인
SELECT relname, n_dead_tup, last_vacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;
```

### 설정 템플릿 (Config)

```sql
-- 커넥션 제한 (RAM 용량에 따라 조절)
ALTER SYSTEM SET max_connections = 100;
ALTER SYSTEM SET work_mem = '8MB';

-- 타임아웃 설정
ALTER SYSTEM SET idle_in_transaction_session_timeout = '30s';
ALTER SYSTEM SET statement_timeout = '30s';

-- 모니터링 확장 도구
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 기본 보안 강화
REVOKE ALL ON SCHEMA public FROM public;

SELECT pg_reload_conf();
```

## 관련 항목

* 에이전트: `database-reviewer` - 전체 데이터베이스 리뷰 워크플로우
* 스킬: `clickhouse-io` - ClickHouse 분석 패턴
* 스킬: `backend-patterns` - API 및 백엔드 아키텍처 패턴

---
*Supabase 에이전트 스킬 기반 (제작: Supabase 팀 / MIT 라이선스)*
