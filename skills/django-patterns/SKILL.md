---
name: django-patterns
description: Django 아키텍처 패턴, DRF를 이용한 REST API 설계, ORM 최선 관행(Best practices), 캐싱, 시그널(Signals), 미들웨어 및 상용 수준의 Django 앱 개발 가이드입니다.
origin: ECC
---

# Django 개발 패턴 (Django Development Patterns)

확장 가능하고 유지보수가 용이한 애플리케이션을 위한 상용 수준의 Django 아키텍처 패턴을 안내합니다.

## 활성화 시점

- Django 웹 애플리케이션 구축 시
- Django REST Framework(DRF) API 설계 시
- Django ORM 및 모델 작업 시
- Django 프로젝트 구조 설정 시
- 캐싱, 시그널, 미들웨어 구현 시

## 프로젝트 구조 (Project Structure)

### 권장 레이아웃

```
myproject/
├── config/                  # 설정 폴더
│   ├── settings/            # 환경별 설정 (base, dev, prod)
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── manage.py
└── apps/                    # 모든 로컬 앱을 보관
    ├── users/
    │   ├── models.py
    │   ├── views.py
    │   ├── serializers.py
    │   ├── services.py      # 비즈니스 로직 분리 (권장)
    │   └── tests/
    └── products/
        └── ...
```

### 설정 분리 패턴 (Split Settings Pattern)

환경별로 설정을 분리하여 관리합니다.

```python
# config/settings/base.py - 공통 설정
INSTALLED_APPS = [
    # ... 기본 앱 ...
    'rest_framework',
    # 로컬 앱
    'apps.users',
    'apps.products',
]

# config/settings/development.py - 개발용
from .base import *
DEBUG = True
DATABASES['default']['NAME'] = 'myproject_dev'

# config/settings/production.py - 상용 서버용
from .base import *
DEBUG = False
ALLOWED_HOSTS = env.list('ALLOWED_HOSTS')
SECURE_SSL_REDIRECT = True
# 보안 관련 설정들...
```

## 모델 설계 패턴 (Model Design Patterns)

### 모델 최선 관행

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    """AbstractUser를 확장한 커스텀 사용자 모델."""
    email = models.EmailField(unique=True)
    # ...

class Product(models.Model):
    """필드 구성이 잘 된 제품 모델 예시."""
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    # ...
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'products'
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['-created_at']),
        ]
```

### QuerySet 최선 관행

커스텀 QuerySet을 사용하여 쿼리 로직을 재사용하고 가독성을 높입니다.

```python
class ProductQuerySet(models.QuerySet):
    def active(self):
        """활성 상태인 제품만 반환."""
        return self.filter(is_active=True)

    def with_category(self):
        """N+1 문제를 방지하기 위해 카테고리를 미리 로드(select_related)."""
        return self.select_related('category')

# 모델에서 사용
class Product(models.Model):
    # ...
    objects = ProductQuerySet.as_manager()

# 사용 예시
Product.objects.active().with_category()
```

## Django REST Framework 패턴

### Serializer 패턴

```python
from rest_framework import serializers

class ProductSerializer(serializers.ModelSerializer):
    """제품 모델 시리얼라이저."""
    category_name = serializers.CharField(source='category.name', read_only=True)
    discount_price = serializers.SerializerMethodField()

    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'discount_price', 'category_name']

    def get_discount_price(self, obj):
        """할인된 가격 계산 로직."""
        return obj.price * 0.9

    def validate_price(self, value):
        """가격 유효성 검사."""
        if value < 0:
            raise serializers.ValidationError("가격은 음수일 수 없습니다.")
        return value
```

### ViewSet 패턴

```python
from rest_framework import viewsets
from .permissions import IsOwnerOrReadOnly

class ProductViewSet(viewsets.ModelViewSet):
    """제품 모델을 위한 ViewSet."""
    queryset = Product.objects.select_related('category').prefetch_related('tags')
    serializer_class = ProductSerializer
    permission_classes = [IsOwnerOrReadOnly]

    def perform_create(self, serializer):
        """생성 시 생성자 정보 자동 저장."""
        serializer.save(created_by=self.request.user)

    @action(detail=False, methods=['get'])
    def featured(self, request):
        """인기 제품 목록 반환 커스텀 액션."""
        featured = self.queryset.filter(is_featured=True)[:10]
        serializer = self.get_serializer(featured, many=True)
        return Response(serializer.data)
```

## 서비스 레이어 패턴 (Service Layer Pattern)

뚱뚱한 모델(Fat Models)이나 뚱뚱한 뷰(Fat Views)를 지양하고, 복잡한 비즈니스 로직은 서비스 레이어로 분리합니다.

```python
# apps/orders/services.py
class OrderService:
    @staticmethod
    @transaction.atomic
    def create_order(user, cart: Cart) -> Order:
        """장바구니로부터 주문을 생성하는 비즈니스 로직."""
        # 주문 생성, 아이템 복사, 장바구니 비우기 등...
        pass
```

## 캐싱 전략 (Caching Strategies)

- **뷰 레벨 캐싱**: `@cache_page(60 * 15)` 데코레이터 사용
- **템플릿 조각 캐싱**: `{% cache 500 sidebar %} ... {% endcache %}` 사용
- **저수준(Low-level) 캐싱**: `cache.get()` 및 `cache.set()`을 직접 사용하여 비싼 연산 결과 저장

## 성능 최적화 (Performance Optimization)

### N+1 쿼리 방지
- **select_related**: Foreign Key나 OneToOne 관계에서 SQL JOIN을 사용하여 한 번에 조회
- **prefetch_related**: ManyToMany나 역참조 관계에서 별도의 쿼리로 데이터를 미리 가져옴

### 인덱싱 (Indexing)
- 자주 필터링되거나 정렬되는 필드에 `db_index=True` 또는 `Meta.indexes` 설정

### 대량 작업 (Bulk Operations)
- `bulk_create`, `bulk_update`, `delete` 등을 활용하여 데이터베이스 요청 횟수 최소화

**기억하십시오**: Django는 많은 편의 기능(Shortcuts)을 제공하지만, 규모가 큰 상용 애플리케이션에서는 구조와 조직화가 간결한 코드보다 더 중요합니다. 유지보수성을 고려하여 설계하십시오.
    
