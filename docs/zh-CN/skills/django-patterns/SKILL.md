---
name: django-patterns
description: Django 아키텍처 패턴, DRF를 사용한 REST API 설계, ORM 베스트 프랙티스, 캐싱, 시그널(Signals), 미들웨어 및 상용 수준의 Django 애플리케이션 구축 가이드입니다.
origin: ECC
---

# Django 개발 패턴

확장 가능하고 유지보수가 용이한 상용 수준의 Django 애플리케이션을 위한 아키텍처 패턴입니다.

## 적용 시점

* Django 웹 애플리케이션을 구축할 때
* Django REST Framework(DRF) 기반 API를 설계할 때
* Django ORM 및 모델을 정의하고 최적화할 때
* Django 프로젝트 구조를 체계적으로 설정할 때
* 캐싱, 시그널, 미들웨어 등을 구현할 때

## 프로젝트 구조

### 권장 레이아웃

```
myproject/
├── config/                  # 프로젝트 설정 폴더
│   ├── __init__.py
│   ├── settings/            # 설정 파일 분리
│   │   ├── __init__.py
│   │   ├── base.py          # 공통 설정
│   │   ├── development.py   # 개발용 설정
│   │   ├── production.py    # 운영용 설정
│   │   └── test.py          # 테스트용 설정
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── manage.py
└── apps/                    # 애플리케이션 모음
    ├── __init__.py
    ├── users/               # 사용자 관련 앱
    │   ├── __init__.py
    │   ├── models.py
    │   ├── views.py
    │   ├── serializers.py
    │   ├── urls.py
    │   ├── permissions.py
    │   ├── filters.py
    │   ├── services.py      # 비즈니스 로직 분리(추천)
    │   └── tests/
    └── products/            # 상품 관련 앱
        └── ...
```

### 설정 파일 분리 패턴 (Split Settings)

```python
# config/settings/base.py
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent.parent

SECRET_KEY = env('DJANGO_SECRET_KEY')
DEBUG = False
ALLOWED_HOSTS = []

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'rest_framework.authtoken',
    'corsheaders',
    # 로컬 앱
    'apps.users',
    'apps.products',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware', # 정적 파일 서빙용
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'config.urls'
WSGI_APPLICATION = 'config.wsgi.application'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': env('DB_NAME'),
        'USER': env('DB_USER'),
        'PASSWORD': env('DB_PASSWORD'),
        'HOST': env('DB_HOST'),
        'PORT': env('DB_PORT', default='5432'),
    }
}

# config/settings/development.py
from .base import *

DEBUG = True
ALLOWED_HOSTS = ['localhost', '127.0.0.1']

DATABASES['default']['NAME'] = 'myproject_dev'

INSTALLED_APPS += ['debug_toolbar']
MIDDLEWARE += ['debug_toolbar.middleware.DebugToolbarMiddleware']

EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# config/settings/production.py
from .base import *

DEBUG = False
ALLOWED_HOSTS = env.list('ALLOWED_HOSTS')
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# 로깅 설정
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'WARNING',
            'class': 'logging.FileHandler',
            'filename': '/var/log/django/django.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'WARNING',
            'propagate': True,
        },
    },
}
```

## 모델 설계 패턴

### 모델 베스트 프랙티스

```python
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.core.validators import MinValueValidator, MaxValueValidator

class User(AbstractUser):
    """AbstractUser를 확장한 커스텀 사용자 모델."""
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=20, blank=True)
    birth_date = models.DateField(null=True, blank=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

    class Meta:
        db_table = 'users'
        verbose_name = 'user'
        verbose_name_plural = 'users'
        ordering = ['-date_joined']

    def __str__(self):
        return self.email

    def get_full_name(self):
        return f"{self.first_name} {self.last_name}".strip()

class Product(models.Model):
    """필드 설정이 최적화된 상품 모델."""
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True, max_length=250)
    description = models.TextField(blank=True)
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        validators=[MinValueValidator(0)]
    )
    stock = models.PositiveIntegerField(default=0)
    is_active = models.BooleanField(default=True)
    category = models.ForeignKey(
        'Category',
        on_delete=models.CASCADE,
        related_name='products' # 역참조 이름 명시
    )
    tags = models.ManyToManyField('Tag', blank=True, related_name='products')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = 'products'
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['-created_at']),
            models.Index(fields=['category', 'is_active']),
        ]
        constraints = [
            models.CheckConstraint(
                check=models.Q(price__gte=0),
                name='price_non_negative'
            )
        ]

    def __str__(self):
        return self.name

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)
```

### QuerySet 베스트 프랙티스 (Custom QuerySet)

```python
from django.db import models

class ProductQuerySet(models.QuerySet):
    """Product 모델을 위한 커스텀 QuerySet."""

    def active(self):
        """활성화된 상품만 반환."""
        return self.filter(is_active=True)

    def with_category(self):
        """N+1 문제를 방지하기 위해 카테고리를 미리 로드."""
        return self.select_related('category')

    def with_tags(self):
        """Many-to-Many 관계의 태그를 미리 로드."""
        return self.prefetch_related('tags')

    def in_stock(self):
        """재고가 있는 상품만 반환."""
        return self.filter(stock__gt=0)

    def search(self, query):
        """이름이나 설명으로 상품 검색."""
        return self.filter(
            models.Q(name__icontains=query) |
            models.Q(description__icontains=query)
        )

class Product(models.Model):
    # ... 필드 정의 ...

    objects = ProductQuerySet.as_manager()  # 커스텀 QuerySet을 매니저로 사용

# 사용 예시
Product.objects.active().with_category().in_stock()
```

### 커스텀 매니저 방법

```python
class ProductManager(models.Manager):
    """복잡한 쿼리를 위한 커스텀 매니저."""

    def get_or_none(self, **kwargs):
        """DoesNotExist 예외 대신 None 반환."""
        try:
            return self.get(**kwargs)
        except self.model.DoesNotExist:
            return None

    def create_with_tags(self, name, price, tag_names):
        """태그와 함께 상품 생성."""
        product = self.create(name=name, price=price)
        tags = [Tag.objects.get_or_create(name=name)[0] for name in tag_names]
        product.tags.set(tags)
        return product

    def bulk_update_stock(self, product_ids, quantity):
        """여러 상품의 재고를 한꺼번에 업데이트."""
        return self.filter(id__in=product_ids).update(stock=quantity)

# 모델에 적용
class Product(models.Model):
    # ... 필드 정의 ...
    custom = ProductManager()
```

## Django REST Framework 패턴

### 시리얼라이저 패턴 (Serializer)

```python
from rest_framework import serializers
from django.contrib.auth.password_validation import validate_password
from .models import Product, User

class ProductSerializer(serializers.ModelSerializer):
    """Product 모델용 시리얼라이저."""

    category_name = serializers.CharField(source='category.name', read_only=True)
    average_rating = serializers.FloatField(read_only=True)
    discount_price = serializers.SerializerMethodField()

    class Meta:
        model = Product
        fields = [
            'id', 'name', 'slug', 'description', 'price',
            'discount_price', 'stock', 'category_name',
            'average_rating', 'created_at'
        ]
        read_only_fields = ['id', 'slug', 'created_at']

    def get_discount_price(self, obj):
        """할인 가격 계산."""
        if hasattr(obj, 'discount') and obj.discount:
            return obj.price * (1 - obj.discount.percent / 100)
        return obj.price

    def validate_price(self, value):
        """가격이 음수가 아닌지 검증."""
        if value < 0:
            raise serializers.ValidationError("가격은 음수일 수 없습니다.")
        return value

class ProductCreateSerializer(serializers.ModelSerializer):
    """상품 생성용 시리얼라이저."""

    class Meta:
        model = Product
        fields = ['name', 'description', 'price', 'stock', 'category']

    def validate(self, data):
        """여러 필드 간의 복합 검증."""
        if data['price'] > 10000 and data['stock'] > 100:
            raise serializers.ValidationError(
                "고가 상품은 대량 재고를 보유할 수 없습니다."
            )
        return data

class UserRegistrationSerializer(serializers.ModelSerializer):
    """회원가입용 시리얼라이저."""

    password = serializers.CharField(
        write_only=True,
        required=True,
        validators=[validate_password],
        style={'input_type': 'password'}
    )
    password_confirm = serializers.CharField(write_only=True, style={'input_type': 'password'})

    class Meta:
        model = User
        fields = ['email', 'username', 'password', 'password_confirm']

    def validate(self, data):
        """비밀번호 일치 여부 검증."""
        if data['password'] != data['password_confirm']:
            raise serializers.ValidationError({
                "password_confirm": "비밀번호가 일치하지 않습니다."
            })
        return data

    def create(self, validated_data):
        """해싱된 비밀번호와 함께 사용자 생성."""
        validated_data.pop('password_confirm')
        password = validated_data.pop('password')
        user = User.objects.create(**validated_data)
        user.set_password(password)
        user.save()
        return user
```

### 뷰셋 패턴 (ViewSet)

```python
from rest_framework import viewsets, status, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated, IsAdminUser
from django_filters.rest_framework import DjangoFilterBackend
from .models import Product
from .serializers import ProductSerializer, ProductCreateSerializer
from .permissions import IsOwnerOrReadOnly
from .filters import ProductFilter
from .services import ProductService

class ProductViewSet(viewsets.ModelViewSet):
    """Product 모델을 위한 ViewSet."""

    queryset = Product.objects.select_related('category').prefetch_related('tags')
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_class = ProductFilter
    search_fields = ['name', 'description']
    ordering_fields = ['price', 'created_at', 'name']
    ordering = ['-created_at']

    def get_serializer_class(self):
        """액션에 따라 적절한 시리얼라이저 반환."""
        if self.action == 'create':
            return ProductCreateSerializer
        return ProductSerializer

    def perform_create(self, serializer):
        """사용자 컨텍스트와 함께 저장."""
        serializer.save(created_by=self.request.user)

    @action(detail=False, methods=['get'])
    def featured(self, request):
        """추천 상품 반환."""
        featured = self.queryset.filter(is_featured=True)[:10]
        serializer = self.get_serializer(featured, many=True)
        return Response(serializer.data)

    @action(detail=True, methods=['post'])
    def purchase(self, request, pk=None):
        """상품 구매 처리."""
        product = self.get_object()
        service = ProductService()
        result = service.purchase(product, request.user)
        return Response(result, status=status.HTTP_201_CREATED)

    @action(detail=False, methods=['get'], permission_classes=[IsAuthenticated])
    def my_products(self, request):
        """현재 사용자가 등록한 상품 목록 반환."""
        products = self.queryset.filter(created_by=request.user)
        page = self.paginate_queryset(products)
        serializer = self.get_serializer(page, many=True)
        return self.get_paginated_response(serializer.data)
```

## 서비스 레이어 패턴 (Service Layer)

```python
# apps/orders/services.py
from typing import Optional
from django.db import transaction
from .models import Order, OrderItem

class OrderService:
    """주문 관련 비즈니스 로직을 담당하는 서비스 레이어."""

    @staticmethod
    @transaction.atomic # 원자적 트랜잭션 보장
    def create_order(user, cart: Cart) -> Order:
        """장바구니로부터 주문 생성."""
        order = Order.objects.create(
            user=user,
            total_price=cart.total_price
        )

        for item in cart.items.all():
            OrderItem.objects.create(
                order=order,
                product=item.product,
                quantity=item.quantity,
                price=item.product.price
            )

        # 장바구니 비우기
        cart.items.all().delete()

        return order

    @staticmethod
    def process_payment(order: Order, payment_data: dict) -> bool:
        """주문 결제 처리."""
        # 결제 게이트웨이(PG) 연동
        payment = PaymentGateway.charge(
            amount=order.total_price,
            token=payment_data['token']
        )

        if payment.success:
            order.status = Order.Status.PAID
            order.save()
            # 확인 이메일 발송
            OrderService.send_confirmation_email(order)
            return True

        return False

    @staticmethod
    def send_confirmation_email(order: Order):
        """주문 확인 이메일 발송 로직."""
        pass
```

## 캐싱 전략

### 뷰 레벨 캐싱
```python
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator

@method_decorator(cache_page(60 * 15), name='dispatch')  # 15분간 캐싱
class ProductListView(generic.ListView):
    model = Product
    template_name = 'products/list.html'
    context_object_name = 'products'
```

### 템플릿 조각 캐싱
```django
{% load cache %}
{% cache 500 sidebar %}
    ... 부하가 큰 사이드바 콘텐츠 ...
{% endcache %}
```

### 로우 레벨 캐싱 (Low-level Cache)
```python
from django.core.cache import cache

def get_featured_products():
    """추천 상품을 캐시와 함께 가져오기."""
    cache_key = 'featured_products'
    products = cache.get(cache_key)

    if products is None:
        products = list(Product.objects.filter(is_featured=True))
        cache.set(cache_key, products, timeout=60 * 15)  # 15분

    return products
```

## 시그널 (Signals)

### 시그널 패턴

```python
# apps/users/signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth import get_user_model
from .models import Profile

User = get_user_model()

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    """사용자 생성 시 자동으로 프로필 생성."""
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    """사용자 저장 시 프로필도 자동 저장."""
    instance.profile.save()

# apps/users/apps.py (앱 설정 파일에서 등록 필요)
from django.apps import AppConfig

class UsersConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'apps.users'

    def ready(self):
        """앱이 준비되었을 때 시그널 임포트."""
        import apps.users.signals
```

## 미들웨어 (Middleware)

### 커스텀 미들웨어

```python
# middleware/active_user_middleware.py
import time
from django.utils.timezone import now
from django.utils.deprecation import MiddlewareMixin

class ActiveUserMiddleware(MiddlewareMixin):
    """활성 사용자를 추적하기 위한 미들웨어."""

    def process_request(self, request):
        if request.user.is_authenticated:
            # 마지막 활동 시간 업데이트
            request.user.last_active = now()
            request.user.save(update_fields=['last_active'])

class RequestLoggingMiddleware(MiddlewareMixin):
    """요청 로깅용 미들웨어."""

    def process_request(self, request):
        request.start_time = time.time()

    def process_response(self, request, response):
        if hasattr(request, 'start_time'):
            duration = time.time() - request.start_time
            logger.info(f'{request.method} {request.path} - {response.status_code} - {duration:.3f}s')
        return response
```

## 성능 최적화

### N+1 쿼리 방지
```python
# ❌ 나쁨 - N+1 쿼리 발생
products = Product.objects.all()
for product in products:
    print(product.category.name)  # 각 상품마다 별도의 쿼리 발생

# ✅ 좋음 - select_related로 단일 쿼리 해결 (ForeignKey, OneToOne)
products = Product.objects.select_related('category').all()
for product in products:
    print(product.category.name)

# ✅ 좋음 - prefetch_related 사용 (ManyToMany, Reverse ForeignKeys)
products = Product.objects.prefetch_related('tags').all()
for product in products:
    for tag in product.tags.all():
        print(tag.name)
```

### 데이터베이스 인덱스
```python
class Meta:
    indexes = [
        models.Index(fields=['name']),
        models.Index(fields=['-created_at']),
        models.Index(fields=['category', 'created_at']), # 복합 인덱스
    ]
```

### 대량 작업 (Bulk Operations)
```python
# 대량 생성 (Bulk create)
Product.objects.bulk_create([
    Product(name=f'Product {i}', price=10.00)
    for i in range(1000)
])

# 대량 업데이트 (Bulk update)
products = Product.objects.all()[:100]
for product in products:
    product.is_active = True
Product.objects.bulk_update(products, ['is_active'])

# 대량 삭제 (Bulk delete)
Product.objects.filter(stock=0).delete()
```

## 빠른 요약 리스트

| 패턴 | 설명 |
|---------|-------------|
| **Split Settings** | 개발/운영/테스트 설정을 철저히 분리 |
| **Custom QuerySet** | 재사용 가능한 조회 로직을 QuerySet 객체에 캡슐화 |
| **Service Layer** | 비즈니스 로직을 뷰에서 분리하여 재사용성 향상 |
| **ViewSet** | 통합된 REST API 엔드포인트 관리 |
| **Serializer Validation** | 요청 및 응답 데이터의 엄격한 검증 및 변환 |
| **select_related** | 외键(FK) 관계 최적화 |
| **prefetch_related** | 대다대(M2M) 관계 최적화 |
| **Cache First** | 부하가 큰 연산은 반드시 캐싱 고려 |
| **Signals** | 이벤트 기반의 비동기적 작업 처리 |
| **Middleware** | 모든 요청/응답에 공통적으로 적용되는 로직 처리 |

**핵심 리마인더**: Django는 수많은 편의 기능을 제공하지만, 상용 애플리케이션에서는 코드의 간결함보다 **구조화된 조직력**이 훨씬 중요합니다. 항상 유지보수성을 염두에 두고 구축하십시오.
