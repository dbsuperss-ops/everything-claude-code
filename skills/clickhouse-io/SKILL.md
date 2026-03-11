---
name: clickhouse-io
description: 고성능 분석 워크로드를 위한 ClickHouse 데이터베이스 패턴, 쿼리 최적화, 분석 및 데이터 엔지니어링 최선 관행(Best practices)입니다.
origin: ECC
---

# ClickHouse 분석 패턴 (ClickHouse Analytics Patterns)

고성능 분석 및 데이터 엔지니어링을 위한 ClickHouse 특화 패턴을 안내합니다.

## 활성화 시점

- ClickHouse 테이블 스키마 설계 시 (MergeTree 엔진 선택 등)
- 분석 쿼리 작성 시 (집계, 윈도우 함수, 조인 등)
- 쿼리 성능 최적화 시 (파티션 프루닝, 프로젝션, 구체화된 뷰(Materialized View) 등)
- 대용량 데이터 적재 시 (배치 인서트, Kafka 통합 등)
- 분석 목적으로 PostgreSQL/MySQL에서 ClickHouse로 마이그레이션할 때
- 실시간 대시보드 또는 시계열 분석 구현 시

## 개요

ClickHouse는 온라인 분석 처리(OLAP)를 위한 컬럼 지향 데이터베이스 관리 시스템(DBMS)입니다. 대규모 데이터셋에 대한 빠른 분석 쿼리에 최적화되어 있습니다.

**주요 특징:**
- 컬럼 지향 저장 방식 (Column-oriented storage)
- 데이터 압축
- 병렬 쿼리 실행
- 분산 쿼리 지원
- 실시간 분석

## 테이블 설계 패턴

### MergeTree 엔진 (가장 일반적임)

```sql
CREATE TABLE markets_analytics (
    date Date,
    market_id String,
    market_name String,
    volume UInt64,
    trades UInt32,
    unique_traders UInt32,
    avg_trade_size Float64,
    created_at DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (date, market_id)
SETTINGS index_granularity = 8192;
```

### ReplacingMergeTree (중복 제거)

```sql
-- 중복 가능성이 있는 데이터(예: 여러 소스에서 오는 데이터)의 경우 사용
CREATE TABLE user_events (
    event_id String,
    user_id String,
    event_type String,
    timestamp DateTime,
    properties String
) ENGINE = ReplacingMergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, event_id, timestamp)
PRIMARY KEY (user_id, event_id);
```

### AggregatingMergeTree (사전 집계)

```sql
-- 집계된 메트릭을 유지하기 위해 사용
CREATE TABLE market_stats_hourly (
    hour DateTime,
    market_id String,
    total_volume AggregateFunction(sum, UInt64),
    total_trades AggregateFunction(count, UInt32),
    unique_users AggregateFunction(uniq, String)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, market_id);

-- 집계된 데이터 조회
SELECT
    hour,
    market_id,
    sumMerge(total_volume) AS volume,
    countMerge(total_trades) AS trades,
    uniqMerge(unique_users) AS users
FROM market_stats_hourly
WHERE hour >= toStartOfHour(now() - INTERVAL 24 HOUR)
GROUP BY hour, market_id
ORDER BY hour DESC;
```

## 쿼리 최적화 패턴

### 효율적인 필터링

```sql
-- ✅ 좋음: 인덱스된 컬럼을 먼저 사용
SELECT *
FROM markets_analytics
WHERE date >= '2025-01-01'
  AND market_id = 'market-123'
  AND volume > 1000
ORDER BY date DESC
LIMIT 100;

-- ❌ 나쁨: 인덱스되지 않은 컬럼을 먼저 필터링
SELECT *
FROM markets_analytics
WHERE volume > 1000
  AND market_name LIKE '%election%'
  AND date >= '2025-01-01';
```

### 집계 (Aggregations)

```sql
-- ✅ 좋음: ClickHouse 전용 집계 함수 사용
SELECT
    toStartOfDay(created_at) AS day,
    market_id,
    sum(volume) AS total_volume,
    count() AS total_trades,
    uniq(trader_id) AS unique_traders,
    avg(trade_size) AS avg_size
FROM trades
WHERE created_at >= today() - INTERVAL 7 DAY
GROUP BY day, market_id
ORDER BY day DESC, total_volume DESC;

-- ✅ 백분위수 계산을 위해 quantile 사용 (percentile보다 효율적임)
SELECT
    quantile(0.50)(trade_size) AS median,
    quantile(0.95)(trade_size) AS p95,
    quantile(0.99)(trade_size) AS p99
FROM trades
WHERE created_at >= now() - INTERVAL 1 HOUR;
```

## 데이터 삽입 패턴

### 배치 인서트 (권장)

```typescript
import { ClickHouse } from 'clickhouse'

// ... 설정 생략 ...

// ✅ 배치 인서트 (효율적)
async function bulkInsertTrades(trades: Trade[]) {
  const values = trades.map(trade => `(
    '${trade.id}',
    '${trade.market_id}',
    '${trade.user_id}',
    ${trade.amount},
    '${trade.timestamp.toISOString()}'
  )`).join(',')

  await clickhouse.query(`
    INSERT INTO trades (id, market_id, user_id, amount, timestamp)
    VALUES ${values}
  `).toPromise()
}

// ❌ 개별 인서트 (매우 느림)
async function insertTrade(trade: Trade) {
  // 루프 안에서 절대로 이렇게 하지 마십시오!
  await clickhouse.query(`
    INSERT INTO trades VALUES ('${trade.id}', ...)
  `).toPromise()
}
```

## 구체화된 뷰 (Materialized Views)

### 실시간 집계

```sql
-- 시간별 통계를 위한 구체화된 뷰 생성
CREATE MATERIALIZED VIEW market_stats_hourly_mv
TO market_stats_hourly
AS SELECT
    toStartOfHour(timestamp) AS hour,
    market_id,
    sumState(amount) AS total_volume,
    countState() AS total_trades,
    uniqState(user_id) AS unique_users
FROM trades
GROUP BY hour, market_id;

-- 구체화된 뷰 조회
SELECT
    hour,
    market_id,
    sumMerge(total_volume) AS volume,
    countMerge(total_trades) AS trades,
    uniqMerge(unique_users) AS users
FROM market_stats_hourly
WHERE hour >= now() - INTERVAL 24 HOUR
GROUP BY hour, market_id;
```

## 성능 모니터링

### 쿼리 성능 확인

```sql
-- 느린 쿼리 체크
SELECT
    query_id,
    user,
    query,
    query_duration_ms,
    read_rows,
    read_bytes,
    memory_usage
FROM system.query_log
WHERE type = 'QueryFinish'
  AND query_duration_ms > 1000
  AND event_time >= now() - INTERVAL 1 HOUR
ORDER BY query_duration_ms DESC
LIMIT 10;
```

## 최선 관행 (Best Practices)

### 1. 파티셔닝 전략
- 시간(보통 월 또는 일) 단위로 파티션을 나눕니다.
- 너무 많은 파티션은 피하십시오 (성능 저하의 원인이 됩니다).
- 파티션 키로는 `DATE` 타입을 사용하십시오.

### 2. 정렬 키 (Ordering Key)
- 가장 자주 필터링되는 컬럼을 앞에 두십시오.
- 카디널리티(Cardinality)를 고려하십시오.
- 정렬 방식은 압축률에 영향을 줍니다.

### 3. 데이터 타입
- 적절한 최소 크기의 타입을 사용하십시오 (UInt32 vs UInt64).
- 반복되는 문자열에는 `LowCardinality`를 사용하십시오.
- 카테고리성 데이터에는 `Enum`을 사용하십시오.

### 4. 피해야 할 것
- `SELECT *` 지양 (필요한 컬럼만 명시)
- `FINAL` 지양 (대신 쿼리 전 데이터 병합 고려)
- 너무 많은 `JOIN` 지양 (분석을 위해 비정규화(Denormalize) 고려)
- 잦은 소량 인서트 지양 (무조건 배치 처리)

**기억하십시오**: ClickHouse는 분석 워크로드에 탁월합니다. 쿼리 패턴에 맞춰 테이블을 설계하고, 배치로 삽입하며, 실시간 집계를 위해 구체화된 뷰를 적극적으로 활용하십시오.
    
