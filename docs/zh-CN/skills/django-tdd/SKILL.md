---
name: django-tdd
description: pytest-django, TDD 방법론, factory_boy, 모킹(Mocking), 커버리지 측정 및 Django REST Framework API 테스트를 포함한 Django 테스트 전략 가이드입니다.
origin: ECC
---

# TDD를 활용한 Django 테스트

pytest, factory_boy 및 Django REST Framework(DRF)를 사용한 테스트 주도 개발(Test-Driven Development) 가이드입니다.

## 적용 시점

* 새로운 Django 애플리케이션을 개발할 때
* Django REST Framework API를 구현하고 검증할 때
* Django 모델, 뷰, 시리얼라이저를 테스트할 때
* Django 프로젝트의 테스트 인프라를 구축할 때

## Django TDD 워크플로우

### Red-Green-Refactor 사이클

```python
# 1단계: RED - 실패하는 테스트 작성
def test_user_creation():
    user = User.objects.create_user(email='test@example.com', password='testpass123')
    assert user.email == 'test@example.com'
    assert user.check_password('testpass123')
    assert not user.is_staff

# 2단계: GREEN - 테스트를 통과시키는 최소한의 코드 작성
# (모델 정의 또는 팩토리 생성 등)

# 3단계: REFACTOR - 테스트 통과 상태를 유지하며 코드 구조 개선
```

## 환경 설정

### pytest 설정

```ini
# pytest.ini
[pytest]
DJANGO_SETTINGS_MODULE = config.settings.test
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts =
    --reuse-db           # 데이터베이스 재사용 (속도 향상)
    --nomigrations       # 마이그레이션 건너뛰기
    --cov=apps           # 커버리지 측정 대상 앱
    --cov-report=html
    --cov-report=term-missing
    --strict-markers
markers =
    slow: 실행이 오래 걸리는 테스트
    integration: 통합 테스트
```

### 테스트 전용 settings 설정

```python
# config/settings/test.py
from .base import *

DEBUG = True # 테스트 에러 확인을 위해 True 설정
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:', # 메모리 DB 사용으로 속도 최적화
    }
}

# 속도를 위해 마이그레이션 비활성화
class DisableMigrations:
    def __contains__(self, item): return True
    def __getitem__(self, item): return None

MIGRATION_MODULES = DisableMigrations()

# 비밀번호 해싱 속도 향상 (보안보다 테스트 속도 중시)
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.MD5PasswordHasher',
]

# 실제 메일 발송 대신 콘솔 출력
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# Celery 작업을 비동기가 아닌 동기 방식으로 즉시 실행
CELERY_TASK_ALWAYS_EAGER = True
CELERY_TASK_EAGER_PROPAGATES = True
```

### conftest.py (공통 픽스처 설정)

```python
# tests/conftest.py
import pytest
from django.contrib.auth import get_user_model

User = get_user_model()

@pytest.fixture(autouse=True)
def timezone_settings(settings):
    """모든 테스트에서 일관된 타임존 사용."""
    settings.TIME_ZONE = 'UTC'

@pytest.fixture
def user(db):
    """기본 테스트 사용자 생성."""
    return User.objects.create_user(
        email='test@example.com',
        password='testpass123',
        username='testuser'
    )

@pytest.fixture
def api_client():
    """DRF API 테스트 클라이언트 반환."""
    from rest_framework.test import APIClient
    return APIClient()

@pytest.fixture
def authenticated_api_client(api_client, user):
    """인증된 API 클라이언트 반환."""
    api_client.force_authenticate(user=user)
    return api_client
```

## Factory Boy (데이터 생성 자동화)

### 팩토리 설정

```python
# tests/factories.py
import factory
from factory import fuzzy
from django.contrib.auth import get_user_model
from apps.products.models import Product, Category

User = get_user_model()

class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = User

    email = factory.Sequence(lambda n: f"user{n}@example.com")
    username = factory.Sequence(lambda n: f"user{n}")
    password = factory.PostGenerationMethodCall('set_password', 'testpass123')
    first_name = factory.Faker('first_name', locale='ko_KR')
    last_name = factory.Faker('last_name', locale='ko_KR')
    is_active = True

class ProductFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = Product

    name = factory.Faker('sentence', nb_words=3)
    slug = factory.LazyAttribute(lambda obj: obj.name.lower().replace(' ', '-'))
    price = fuzzy.FuzzyDecimal(10.00, 1000.00, 2)
    stock = fuzzy.FuzzyInteger(0, 100)
    category = factory.SubFactory('tests.factories.CategoryFactory')
```

### 팩토리 활용 예시

```python
# tests/test_models.py
def test_product_creation(db):
    """팩토리를 사용한 상품 생성 테스트."""
    product = ProductFactory(price=100.00, stock=50)
    assert product.price == 100.00
    assert product.stock == 50

def test_multiple_products(db):
    """여러 개의 상품 대량 생성 테스트."""
    products = ProductFactory.create_batch(10)
    assert len(products) == 10
```

## 모델 테스트

```python
# tests/test_models.py
@pytest.mark.django_db
class TestProductModel:
    def test_product_slug_generation(self):
        """자동 슬러그 생성 테스트."""
        product = ProductFactory(name='Test Product')
        assert product.slug == 'test-product'

    def test_product_price_validation(self):
        """가격 음수 검증 테스트."""
        from django.core.exceptions import ValidationError
        product = ProductFactory(price=-10)
        with pytest.raises(ValidationError):
            product.full_clean()
```

## DRF API 테스트

```python
# tests/test_api.py
import pytest
from rest_framework import status
from django.urls import reverse

@pytest.mark.django_db
class TestProductAPI:
    def test_list_products(self, api_client):
        """상품 목록 조회 API 테스트."""
        ProductFactory.create_batch(5)
        url = reverse('api:product-list')
        response = api_client.get(url)
        assert response.status_code == status.HTTP_200_OK
        assert response.data['count'] == 5

    def test_create_product_authorized(self, authenticated_api_client):
        """인증된 사용자의 상품 생성 API 테스트."""
        url = reverse('api:product-list')
        data = {'name': 'New Product', 'price': '99.99', 'stock': 10}
        response = authenticated_api_client.post(url, data)
        assert response.status_code == status.HTTP_201_CREATED
```

## 모킹 (Mocking) 및 패칭 (Patching)

### 외부 서비스 모킹 (Stripe 등)

```python
from unittest.mock import patch

@patch('apps.payments.services.stripe.Charge.create')
def test_successful_payment(self, mock_charge, authenticated_api_client, product):
    """Stripe 결제 성공 시나리오 모킹 테스트."""
    mock_charge.return_value = {'id': 'ch_123', 'status': 'succeeded'}
    
    url = reverse('api:purchase', kwargs={'pk': product.id})
    response = authenticated_api_client.post(url, {'token': 'tok_visa'})
    
    assert response.status_code == status.HTTP_201_CREATED
    mock_charge.assert_called_once()
```

### 이메일 발송 테스트

```python
from django.core import mail

def test_order_email(db, order):
    """주문 완료 시 메일 발송 여부 테스트."""
    order.send_confirmation_email()
    assert len(mail.outbox) == 1
    assert '주문 확인' in mail.outbox[0].subject
```

## 테스트 베스트 프랙티스

### 권장 사항 (DO)
* **팩토리(Factory) 사용**: 직접 객체를 생성하지 말고 Factory Boy를 활용하십시오.
* **하나의 단언(One Assertion)**: 테스트 하나는 하나의 목적에만 집중하십시오.
* **명확한 테스트 이름**: `test_user_cannot_delete_others_post`와 같이 의도를 드러내십시오.
* **경계값 테스트**: 빈 입력, None, 최대치 등 엣지 케이스를 테스트하십시오.
* **외부 서비스 격리**: 실제 API를 호출하지 말고 모킹(Mock)하십시오.
* **속도 유지**: `--reuse-db`, `--nomigrations`를 활용하여 피드백 사이클을 단축하십시오.

### 금지 사항 (DON'T)
* **Django 내부 로직 테스트**: Django 프레임워크 자체의 동작은 신뢰하십시오.
* **서드파티 라이브러리 테스트**: 외부 라이브러리는 이미 테스트되었다고 가정하십시오.
* **실패 무시**: 모든 테스트는 반드시 통과해야 합니다.
* **테스트 간 의존성**: 각 테스트는 독립적으로, 어떤 순서로든 실행 가능해야 합니다.
* **운영 DB 사용**: 테스트 중 데이터가 손실될 수 있으므로 전용 테스트 DB만 사용하십시오.

## 커버리지 (Coverage)

```bash
# 커버리지와 함께 테스트 실행
pytest --cov=apps --cov-report=term-missing

# 권장 커버리지 목표
# - 모델: 90%+
# - 서비스/비즈니스 로직: 90%+
# - 시리얼라이저: 85%+
# - 뷰/컨트롤러: 80%+
```

## 빠른 요약 리스트

| 패턴/도구 | 용도 |
|---------|-------|
| `@pytest.mark.django_db` | 데이터베이스 접근 권한 부여 |
| `client` | Django 기본 웹 테스트 클라이언트 |
| `api_client` | DRF 전용 API 테스트 클라이언트 |
| `factory.create_batch(n)` | 데이터를 n개 일괄 생성 |
| `patch('module.func')` | 외부 의존성을 모의 객체로 교체 |
| `override_settings` | 테스트 중에만 일시적으로 settings 변경 |
| `mail.outbox` | 전송된 이메일 목록 확인 |

**핵심 리마인더**: 테스트는 곧 **문서**입니다. 잘 작성된 테스트는 코드가 어떻게 동작해야 하는지를 가장 정확하게 설명해 줍니다. 테스트를 단순하고 읽기 쉽게 유지하십시오.
