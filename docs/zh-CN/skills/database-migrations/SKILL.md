---
name: database-migrations
description: 데이터베이스 마이그레이션 베스트 프랙티스입니다. 스키마 변경, 데이터 이관, 롤백 및 무중단 배포 전략을 포함하며 PostgreSQL, MySQL 및 주요 ORM(Prisma, Drizzle, Django, TypeORM, golang-migrate)을 다룹니다.
origin: ECC
---

# 데이터베이스 마이그레이션 패턴

운영 시스템을 위한 안전하고 가역적인(Reversible) 데이터베이스 스키마 변경 전략을 제공합니다.

## 적용 시점

* 데이터베이스 테이블을 생성하거나 수정할 때
* 컬럼이나 인덱스를 추가/삭제할 때
* 데이터 이관(Data migration), 데이터 백필(Backfill), 데이터 변환 작업을 수행할 때
* 무중단(Zero-downtime) 스키마 변경을 계획할 때
* 새 프로젝트에 마이그레이션 도구를 설정할 때

## 핵심 원칙

1. **모든 변경은 마이그레이션으로 관리** — 운영 데이터베이스를 수동으로 직접 수정하지 마십시오.
2. **운영 환경에서는 전진(Forward) 마이그레이션만 수행** — 롤백이 필요한 경우에도 새로운 전진 마이그레이션을 통해 복구하십시오.
3. **스키마와 데이터 마이그레이션 분리** — 하나의 마이그레이션 파일에 DDL(스키마 변경)과 DML(데이터 조작)을 섞지 마십시오.
4. **운영 환경 수준의 데이터로 테스트** — 100행 규모에서 성공한 마이그레이션이 1,000만 행 규모에서는 테이블 락(Lock)을 유발할 수 있습니다.
5. **배포된 마이그레이션은 불변(Immutable)** — 이미 운영 환경에 적용된 마이그레이션 파일을 수정하지 마십시오.

## 마이그레이션 안전 체크리스트

마이그레이션 적용 전 확인 사항:

* [ ] UP(적용)과 DOWN(취소) 로직이 모두 포함되었는가(또는 명시적으로 취소 불가로 표시했는가).
* [ ] 대규모 테이블에 전체 테이블 락(Full table lock)을 걸지 않는가(동시성 옵션 사용 등).
* [ ] 새로운 컬럼에 기본값(Default)이 있거나 Null 허용(Nullable)인가(기존 데이터가 있는 경우 기본값 없는 NOT NULL 추가 금지).
* [ ] 인덱스를 동시성 모드(`CONCURRENTLY`)로 생성하는가(기존 테이블의 경우 `CREATE TABLE` 내에 인덱스를 포함하지 말 것).
* [ ] 데이터 백필 작업이 스키마 변경과 별도의 마이그레이션으로 분리되었는가.
* [ ] 운영 데이터의 복사본(Staging 등)에서 테스트를 완료했는가.
* [ ] 롤백 계획이 문서화되어 있는가.

## PostgreSQL 패턴

### 안전한 컬럼 추가

```sql
-- 좋음: Null 허용 컬럼 추가, 락 발생 없음
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- 좋음: 기본값이 있는 컬럼 추가 (Postgres 11+ 버전은 즉시 반영되며 테이블 재작성이 없음)
ALTER TABLE users ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT true;

-- 나쁨: 기존 데이터가 있는 테이블에 기본값 없이 NOT NULL 컬럼 추가 (전체 재작성 필요)
ALTER TABLE users ADD COLUMN role TEXT NOT NULL;
-- 이는 테이블 전체에 락을 걸고 모든 행을 재작성합니다.
```

### 무중단 인덱스 추가

```sql
-- 나쁨: 대규모 테이블에서 쓰기 작업을 차단함
CREATE INDEX idx_users_email ON users (email);

-- 좋음: 논블로킹(Non-blocking) 방식, 동시 쓰기 허용
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);

-- 참고: CONCURRENTLY 옵션은 트랜잭션 블록 내에서 실행될 수 없습니다.
-- 대부분의 마이그레이션 도구에서 이를 위해 별도의 처리가 필요합니다.
```

### 컬럼 이름 변경 (무중단 전략)
운영 중인 DB에서 컬럼명을 직접 바꾸지 마십시오. '확장-축소(Expand-Contract)' 패턴을 사용합니다:

1. **새 컬럼 추가** (마이그레이션 001): `display_name` 추가
2. **데이터 백필** (마이그레이션 002, 데이터 마이그레이션): `username` 값을 `display_name`으로 복사
3. **애플리케이션 코드 업데이트**: 두 컬럼 모두에 읽기/쓰기를 수행하도록 수정하여 배포
4. **이전 컬럼 삭제** (마이그레이션 003): 애플리케이션에서 더 이상 사용하지 않는 `username` 삭제

### 안전한 컬럼 삭제
1. 애플리케이션 코드에서 해당 컬럼에 대한 모든 참조를 제거합니다.
2. 해당 코드를 배포합니다.
3. 다음 마이그레이션에서 컬럼을 삭제합니다:
   ```sql
   ALTER TABLE orders DROP COLUMN legacy_status;
   ```
   *Django의 경우: `SeparateDatabaseAndState`를 사용하여 DB에서 먼저 비활성화한 후 다음 마이그레이션에서 삭제하는 방식을 권장합니다.*

### 대규모 데이터 마이그레이션 (Batch Update)

```sql
-- 나쁨: 하나의 트랜잭션에서 모든 행 수정 (테이블 락 유발)
UPDATE users SET normalized_email = LOWER(email);

-- 좋음: 배치 처리를 통한 점진적 업데이트
DO $$
DECLARE
  batch_size INT := 10000;
  rows_updated INT;
BEGIN
  LOOP
    UPDATE users
    SET normalized_email = LOWER(email)
    WHERE id IN (
      SELECT id FROM users
      WHERE normalized_email IS NULL
      LIMIT batch_size
      FOR UPDATE SKIP LOCKED
    );
    GET DIAGNOSTICS rows_updated = ROW_COUNT;
    RAISE NOTICE '업데이트된 행 수: %', rows_updated;
    EXIT WHEN rows_updated = 0;
    COMMIT;
  END LOOP;
END $$;
```

## Prisma (TypeScript/Node.js)

### 워크플로우
```bash
# 스키마 변경사항으로부터 마이그레이션 생성
npx prisma migrate dev --name add_user_avatar

# 운영 환경에 대기 중인 마이그레이션 적용
npx prisma migrate deploy

# 데이터베이스 초기화 (개발 환경 전용)
npx prisma migrate reset

# 스키마 변경 후 클라이언트 생성
npx prisma generate
```

### 모델 예시
```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  avatarUrl String?  @map("avatar_url")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  orders    Order[]

  @@map("users")
  @@index([email])
}
```

### 커스텀 SQL 마이그레이션
Prisma로 직접 표현하기 어려운 작업(CONCURRENT 인덱스, 데이터 백필 등)의 경우:
```bash
# 파일만 생성하고 SQL은 수동으로 편집
npx prisma migrate dev --create-only --name add_email_index
```

```sql
-- migrations/20240115_add_email_index/migration.sql
-- Prisma는 CONCURRENTLY를 직접 생성하지 못하므로 수동으로 작성합니다.
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_users_email ON users (email);
```

## Drizzle (TypeScript/Node.js)

### 워크플로우
```bash
# 스키마 변경사항으로부터 마이그레이션 생성
npx drizzle-kit generate

# 마이그레이션 적용
npx drizzle-kit migrate

# 스키마를 즉시 푸시 (개발 환경 전용, 마이그레이션 파일 없음)
npx drizzle-kit push
```

## Django (Python)

### 워크플로우
```bash
# 모델 변경사항으로부터 마이그레이션 생성
python manage.py makemigrations

# 마이그레이션 적용
python manage.py migrate

# 상태 확인
python manage.py showmigrations

# 커스텀 SQL용 빈 마이그레이션 생성
python manage.py makemigrations --empty app_name -n description
```

### 데이터 마이그레이션 예시
```python
from django.db import migrations

def backfill_display_names(apps, schema_editor):
    User = apps.get_model("accounts", "User")
    batch_size = 5000
    users = User.objects.filter(display_name="")
    while users.exists():
        batch = list(users[:batch_size])
        for user in batch:
            user.display_name = user.username
        User.objects.bulk_update(batch, ["display_name"], batch_size=batch_size)

def reverse_backfill(apps, schema_editor):
    pass  # 데이터 마이그레이션의 경우 되돌리기가 필요 없으면 비워둠

class Migration(migrations.Migration):
    dependencies = [("accounts", "0015_add_display_name")]

    operations = [
        migrations.RunPython(backfill_display_names, reverse_backfill),
    ]
```

## golang-migrate (Go)

### 워크플로우
```bash
# 마이그레이션 파일 쌍(UP/DOWN) 생성
migrate create -ext sql -dir migrations -seq add_user_avatar

# 모든 마이그레이션 적용
migrate -path migrations -database "$DATABASE_URL" up

# 마지막 마이그레이션 취소
migrate -path migrations -database "$DATABASE_URL" down 1

# 버전 강제 지정 (Dirty 상태 복구용)
migrate -path migrations -database "$DATABASE_URL" force VERSION_NUMBER
```

## 무중단 마이그레이션 전략 (Expand-Contract)

주요 운영 환경 변경 시 다음 단계를 준수하십시오:

1. **확장(EXPAND)** 단계:
   - 새로운 컬럼이나 테이블을 추가합니다(Null 허용 혹은 기본값 설정).
   - 배포: 애플리케이션이 이전 컬럼과 새 컬럼 **모두**에 데이터를 기록하도록 합니다.
   - 기존 데이터를 새 컬럼으로 백필합니다.

2. **이관(MIGRATE)** 단계:
   - 배포: 애플리케이션이 데이터를 읽을 때는 **새 컬럼**을 사용하고, 쓸 때는 여전히 **두 곳 모두**에 기록합니다.
   - 데이터 일관성을 최종 확인합니다.

3. **축소(CONTRACT)** 단계:
   - 배포: 애플리케이션이 **새 컬럼만** 사용하도록 수정합니다.
   - 별도의 마이그레이션을 통해 더 이상 쓰지 않는 이전 컬럼/테이블을 삭제합니다.

## 피해야 할 안티 패턴

| 안티 패턴 | 문제점 | 해결 방법 |
|-------------|-------------|-----------------|
| 운영 DB 직접 수정 | 이력 관리가 안 되고 재현 불가능함 | 항상 마이그레이션 파일을 통해 실행 |
| 배포된 마이그레이션 수정 | 환경 간 데이터베이스 상태 불일치 발생 | 새로운 마이그레이션 파일을 작성 |
| 기본값 없는 NOT NULL 추가 | 테이블 락 발생 및 모든 행 재작성 유발 | Null 허용으로 추가 후 데이터 백필, 그 다음 제약 조건 추가 |
| 대규모 테이블에 단일 인덱스 생성 | 인덱스 생성 중 쓰기 작업 차단 | `CREATE INDEX CONCURRENTLY` 사용 |
| 스키마와 데이터 변경을 섞음 | 롤백이 어렵고 트랜잭션 시간이 길어짐 | 스키마 변경과 데이터 변경 마이그레이션 분리 |
| 코드 수정 전 데이터 삭제 | 컬럼 부재로 인한 애플리케이션 런타임 에러 | 코드에서 먼저 제거하고 다음 배포 때 DB 컬럼 삭제 |
