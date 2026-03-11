---
name: clickhouse-io
description: ClickHouse 데이터베이스 스키마, 쿼리 최적화, 분석 및 고성능 분석 워크로드를 위한 데이터 엔지니어링 베스트 프랙티스입니다.
origin: ECC
---

# ClickHouse 분석 패턴

고성능 분석 및 데이터 엔지니어링을 위한 ClickHouse 특화 패턴입니다.

## 적용 시점

* ClickHouse 테이블 구조를 설계할 때 (MergeTree 엔진 선택 등)
* 분석 쿼리(집계, 윈도우 함수, 조인 등)를 작성할 때
* 쿼리 성능을 최적화할 때 (파티션 프루닝, 프로젝션, 구체화된 뷰 활용)
* 대량의 데이터를 수집할 때 (배치 삽입, Kafka 연동 등)
* 분석 목적으로 PostgreSQL/MySQL에서 ClickHouse로 데이터를 마이그레이션할 때
* 실시간 대시보드 또는 시계열 데이터 분석을 구현할 때

## 개요

ClickHouse는 온라인 분석 처리(OLAP)를 위한 컬럼 지향 데이터베이스 관리 시스템(DBMS)입니다. 대규모 데이터셋에 대한 빠른 분석 쿼리 수행에 최적화되어 있습니다.

**주요 특징:**

* 컬럼 지향 저장 방식 (Columnar Storage)
* 강력한 데이터 압축
* 병렬 쿼리 실행
* 분산 쿼리 지원
* 실시간 분석 가능

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
-- 여러 소스에서 데이터가 유입되어 중복이 발생할 수 있는 경우 사용
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
-- 집계된 메트릭을 유지 관리할 때 사용
CREATE TABLE market_stats_hourly (
    hour DateTime,
    market_id String,
    total_volume AggregateFunction(sum, UInt64),
    total_trades AggregateFunction(count, UInt32),
    unique_users AggregateFunction(uniq, String)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, market_id);

-- 집계된 데이터 쿼리 예시
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
-- ✅ 좋은 예: 인덱싱된 컬럼을 먼저 조건에 사용
SELECT *
FROM markets_analytics
WHERE date >= '2025-01-01'
  AND market_id = 'market-123'
  AND volume > 1000
ORDER BY date DESC
LIMIT 100;

-- ❌ 나쁜 예: 인덱싱되지 않은 컬럼을 먼저 필터링
SELECT *
FROM markets_analytics
WHERE volume > 1000
  AND market_name LIKE '%election%'
  AND date >= '2025-01-01';
```

### 집계 (Aggregation)

```sql
-- ✅ 좋은 예: ClickHouse 전용 집계 함수 사용
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

-- ✅ 백분위수(Percentiles) 계산 시 quantile 함수 사용 (효율적임)
SELECT
    quantile(0.50)(trade_size) AS median,
    quantile(0.95)(trade_size) AS p95,
    quantile(0.99)(trade_size) AS p99
FROM trades
WHERE created_at >= now() - INTERVAL 1 HOUR;
```

### 윈도우 함수 (Window Functions)

```sql
-- 누적 합계(Running totals) 계산
SELECT
    date,
    market_id,
    volume,
    sum(volume) OVER (
        PARTITION BY market_id
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_volume
FROM markets_analytics
WHERE date >= today() - INTERVAL 30 DAY
ORDER BY market_id, date;
```

## 데이터 삽입 패턴

### 배치 삽입 (권장)

```typescript
import { ClickHouse } from 'clickhouse'

const clickhouse = new ClickHouse({
  url: process.env.CLICKHOUSE_URL,
  port: 8123,
  basicAuth: {
    username: process.env.CLICKHOUSE_USER,
    password: process.env.CLICKHOUSE_PASSWORD
  }
})

// ✅ 효율적인 배치 삽입
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

// ❌ 나쁜 예: 개별 행 단위 삽입 (매우 느림)
async function insertTrade(trade: Trade) {
  // 루프 안에서 이런 식으로 처리하지 마십시오!
  await clickhouse.query(`
    INSERT INTO trades VALUES ('${trade.id}', ...)
  `).toPromise()
}
```

### 스트리밍 삽입

```typescript
// 지속적인 데이터 수집을 위한 스트리밍 방식
import { createWriteStream } from 'fs'
import { pipeline } from 'stream/promises'

async function streamInserts() {
  const stream = clickhouse.insert('trades').stream()

  for await (const batch of dataSource) {
    stream.write(batch)
  }

  await stream.end()
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

-- 구체화된 뷰 쿼리 예시
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
-- 느린 쿼리 확인
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

### 테이블 통계 확인

```sql
-- 테이블 크기 확인
SELECT
    database,
    table,
    formatReadableSize(sum(bytes)) AS size,
    sum(rows) AS rows,
    max(modification_time) AS latest_modification
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY sum(bytes) DESC;
```

## 일반적인 분석 쿼리 예시

### 시계열 분석

```sql
-- 일일 활성 사용자(DAU)
SELECT
    toDate(timestamp) AS date,
    uniq(user_id) AS daily_active_users
FROM events
WHERE timestamp >= today() - INTERVAL 30 DAY
GROUP BY date
ORDER BY date;

-- 잔存(Retention) 분석
SELECT
    signup_date,
    countIf(days_since_signup = 0) AS day_0,
    countIf(days_since_signup = 1) AS day_1,
    countIf(days_since_signup = 7) AS day_7,
    countIf(days_since_signup = 30) AS day_30
FROM (
    SELECT
        user_id,
        min(toDate(timestamp)) AS signup_date,
        toDate(timestamp) AS activity_date,
        dateDiff('day', signup_date, activity_date) AS days_since_signup
    FROM events
    GROUP BY user_id, activity_date
)
GROUP BY signup_date
ORDER BY signup_date DESC;
```

### 퍼널(Funnel) 분석

```sql
-- 전환 퍼널
SELECT
    countIf(step = 'viewed_market') AS viewed,
    countIf(step = 'clicked_trade') AS clicked,
    countIf(step = 'completed_trade') AS completed,
    round(clicked / viewed * 100, 2) AS view_to_click_rate,
    round(completed / clicked * 100, 2) AS click_to_completion_rate
FROM (
    SELECT
        user_id,
        session_id,
        event_type AS step
    FROM events
    WHERE event_date = today()
)
GROUP BY session_id;
```

### 코호트(Cohort) 분석

```sql
-- 가입 월별 사용자 코호트
SELECT
    toStartOfMonth(signup_date) AS cohort,
    toStartOfMonth(activity_date) AS month,
    dateDiff('month', cohort, month) AS months_since_signup,
    count(DISTINCT user_id) AS active_users
FROM (
    SELECT
        user_id,
        min(toDate(timestamp)) OVER (PARTITION BY user_id) AS signup_date,
        toDate(timestamp) AS activity_date
    FROM events
)
GROUP BY cohort, month, months_since_signup
ORDER BY cohort, months_since_signup;
```

## 데이터 파이프라인 패턴

### ETL 패턴

```typescript
// 추출(Extract), 변환(Transform), 로드(Load)
async function etlPipeline() {
  // 1. 소스에서 추출
  const rawData = await extractFromPostgres()

  // 2. 변환
  const transformed = rawData.map(row => ({
    date: new Date(row.created_at).toISOString().split('T')[0],
    market_id: row.market_slug,
    volume: parseFloat(row.total_volume),
    trades: parseInt(row.trade_count)
  }))

  // 3. ClickHouse로 로드
  await bulkInsertToClickHouse(transformed)
}

// 매 시간 주기적으로 실행
setInterval(etlPipeline, 60 * 60 * 1000)
```

### 변경 데이터 캡처 (CDC)

```typescript
// PostgreSQL의 변경 사항을 감지하여 ClickHouse에 동기화
import { Client } from 'pg'

const pgClient = new Client({ connectionString: process.env.DATABASE_URL })

pgClient.query('LISTEN market_updates')

pgClient.on('notification', async (msg) => {
  const update = JSON.parse(msg.payload)

  await clickhouse.insert('market_updates', [
    {
      market_id: update.id,
      event_type: update.operation,  // INSERT, UPDATE, DELETE
      timestamp: new Date(),
      data: JSON.stringify(update.new_data)
    }
  ])
})
```

## 베스트 프랙티스 요약

### 1. 파티션 전략
* 시간 단위(보통 월 또는 일)로 파티션을 구성하십시오.
* 파티션이 너무 많아지면 성능이 저하되므로 주의하십시오.
* 파티션 키에는 `DATE` 타입을 사용하십시오.

### 2. 정렬 키 (Sorting Keys)
* 필터링 조건에 가장 자주 쓰이는 컬럼을 앞에 두십시오.
* 카디널리티가 높은 컬럼을 우선 고려하십시오.
* 정렬 상태는 데이터 압축 효율에 영향을 줍니다.

### 3. 데이터 타입 선택
* 가능한 가장 작은 적절한 타입을 사용하십시오 (UInt32 vs UInt64).
* 반복되는 문자열에는 `LowCardinality`를 사용하십시오.
* 범주형 데이터에는 `Enum` 타입을 사용하십시오.

### 4. 피해야 할 사항
* `SELECT *` 사용 지양 (필요한 컬럼만 명시)
* `FINAL` 구문 과용 금지 (대신 쿼리 전 데이터 병합 고려)
* 과도한 `JOIN` 사용 지양 (분석용은 반정규화 권장)
* 빈번한 소량의 삽입 금지 (반드시 배치 처리)

### 5. 모니터링
* 쿼리 성능을 정기적으로 추적하십시오.
* 디스크 사용량을 모니터링하십시오.
* 병합(Merge) 작업 현황을 확인하십시오.
* 느린 쿼리 로그를 수시로 점검하십시오.

**핵심**: ClickHouse는 분석 워크로드에 최적화되어 있습니다. 쿼리 패턴에 맞춰 테이블을 설계하고, 대량으로 삽입하며, 구체화된 뷰를 적극적으로 활용하여 실시간 집계 성능을 확보하십시오.
