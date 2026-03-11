---
description: PostgreSQL과 Celery를 사용하는 Django REST Framework(DRF) 기반 API 프로젝트의 CLAUDE.md 설정 예시입니다.
---

# Django REST API 프로젝트 가이드 (CLAUDE.md 예시)

이 파일은 Django REST Framework 기반의 백엔드 프로젝트에서 Claude Code가 준수해야 할 핵심 지침을 담고 있습니다.

## 프로젝트 개요

**기술 스택:** Python 3.12+, Django 5.x, Django REST Framework, PostgreSQL, Celery + Redis, pytest, Docker Compose.

**아키텍처:** 도메인 주도 설계(DDD)를 채택하며, 각 비즈니스 도메인은 개별 Django 앱으로 관리합니다. 모든 엔드포인트는 JSON으로 통신하며 템플릿 렌더링은 사용하지 않습니다.

## 핵심 규칙

### 1. Python 코딩 관행
* **타입 힌트**: 모든 함수 시그니처에 타입 힌트를 명시하십시오. (`from __future__ import annotations` 사용 권장)
* **로그 관리**: `print()` 대신 `logging.getLogger(__name__)`을 사용하십시오.
* **문자열**: f-strings만 사용하며 `%`나 `.format()`은 지양하십시오.
* **경로 처리**: `os.path` 대신 `pathlib.Path`를 사용하십시오.
* **임포트 정렬**: 표준 라이브러리, 제3자 라이브러리, 로컬 패키지 순으로 정렬하십시오. (ruff로 강제)

### 2. 데이터베이스 및 ORM
* **ORM 우선**: 모든 쿼리는 Django ORM을 사용하십시오. 로우 SQL은 파라미터화된 `.raw()`에서만 허용됩니다.
* **마이그레이션**: 마이그레이션 파일은 반드시 Git에 포함하며, 운영 환경에서 `--fake` 옵션은 지양하십시오.
* **성능 최적화**: N+1 쿼리 방지를 위해 `select_related()`와 `prefetch_related()`를 적극 활용하십시오.
* **모델 필수 필드**: 모든 모델은 `created_at` 및 `updated_at` 자동 생성 필드를 포함해야 합니다.
* **인덱스**: `filter()`, `order_by()` 또는 `WHERE` 절에 자주 쓰이는 필드에 인덱스를 설정하십시오.

### 3. 인증 및 권한 (Auth)
* **JWT 사용**: `simplejwt`를 활용한 Access/Refresh 토큰 방식을 구현하십시오.
* **권한 설정**: 각 뷰마다 명시적인 권한 클래스를 설정하십시오. (기본 설정에 의존 금지)
* **블랙리스트**: 로그아웃 시 토큰 블랙리스트 기능을 활성화하십시오.

### 4. 시리어라이저 (Serializers)
* **검증 위치**: 비즈니스 로직 검증은 뷰(View)가 아닌 시리어라이저 레벨에서 수행하여 뷰를 간결하게 유지하십시오.
* **읽기/쓰기 분리**: 입력과 출력 구조가 다를 경우 읽기용과 쓰기용 시리어라이저를 분리하십시오.

## 권장 파일 구조

```text
config/               # 프로젝트 설정 (base, local, production)
apps/
  accounts/           # 사용자 인증, 프로필 등
    services.py       # 비즈니스 로직 레이어
    tests/            # pytest 기반 테스트
  orders/             # 주문 관리 (tasks.py 포함)
  products/           # 상품 카탈로그
core/                 # 공통 예외, 권한, 미들웨어, 유틸리티
```

## 주요 패턴 예시

### 서비스 레이어 (비즈니스 로직)
```python
@transaction.atomic
def create_order(*, customer, product_id: uuid.UUID, quantity: int) -> Order:
    # 재고 검증 및 주문 생성 로직
    # Celery 태스크 호출: send_order_confirmation.delay(order.id)
    return order
```

### 테스트 패턴 (pytest + Factory Boy)
```python
@pytest.mark.django_db
class TestCreateOrder:
    def test_create_order_success(self):
        # APIClient와 Factory를 사용한 기능 검증
        assert response.status_code == 201
```

## 주요 명령어 (Slash Commands)

* `/plan`: 신규 기능 개발 계획(예: 결제 시스템 연동) 수립
* `/tdd`: pytest 기반의 테스트 주도 개발 워크플로우
* `/python-review`: 파이썬 특유의 코드 리뷰 및 보안 스캔
* `/verify`: 빌드, 린트, 테스트 및 전체 보안 점검

**핵심**: 프로젝트의 일관성을 위해 도메인별 관심사 분리를 엄격히 준수하고, 모든 비즈니스 로직은 테스트 코드로 보호되어야 합니다.
