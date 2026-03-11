---
name: django-security
description: Django 보안 베스트 프랙티스, 인증, 인가, CSRF 방지, SQL 인젝션 예방, XSS 방지 및 보안 배포 설정 가이드입니다.
origin: ECC
---

# Django 보안 베스트 프랙티스

주요 취약점으로부터 Django 애플리케이션을 보호하기 위한 종합 보안 가이드입니다.

## 적용 시점

* Django 인증(Authentication) 및 인가(Authorization)를 설정할 때
* 사용자 권한(Permissions) 및 그룹(Roles)을 구현할 때
* 운영 환경용 보안 설정을 구성할 때
* Django 애플리케이션의 보안 취약점을 검토할 때
* 실제 운영 환경에 배포하기 직전 점검용으로 활용

## 핵심 보안 설정

### 운영 환경(Production) 설정 구성

```python
# settings/production.py
import os
from django.core.exceptions import ImproperlyConfigured

DEBUG = False  # ⚠️ 매우 중요: 운영 환경에서는 절대 True로 두지 마십시오.

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# 보안 헤더 설정
SECURE_SSL_REDIRECT = True # 모든 HTTP 요청을 HTTPS로 리다이렉트
SESSION_COOKIE_SECURE = True # 세션 쿠키를 HTTPS에서만 전송
CSRF_COOKIE_SECURE = True # CSRF 쿠키를 HTTPS에서만 전송
SECURE_HSTS_SECONDS = 31536000  # 1년 (HSTS 활성화)
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_CONTENT_TYPE_NOSNIFF = True # MIME 스니핑 방지
SECURE_BROWSER_XSS_FILTER = True # 브라우저 XSS 필터 활성화
X_FRAME_OPTIONS = 'DENY' # 클릭재킹 방지

# 쿠키 보안 강화
SESSION_COOKIE_HTTPONLY = True # 자바스크립트의 세션 쿠키 접근 방지
CSRF_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
CSRF_COOKIE_SAMESITE = 'Lax'

# 시크릿 키 (반드시 환경 변수를 통해 주입)
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
if not SECRET_KEY:
    raise ImproperlyConfigured('DJANGO_SECRET_KEY 환경 변수가 설정되지 않았습니다.')

# 비밀번호 검증 정책
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 12, # 최소 12자 이상 권장
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

## 인증 (Authentication)

### 커스텀 사용자 모델
기본 모델보다 보안성이 높은 커스텀 유저 모델 사용을 권장합니다.

```python
# apps/users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    """보안 및 확장성을 고려한 커스텀 사용자 모델."""
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=20, blank=True)

    USERNAME_FIELD = 'email'  # 이메일을 아이디로 사용
    REQUIRED_FIELDS = ['username']

    class Meta:
        db_table = 'users'
        verbose_name = 'User'
        verbose_name_plural = 'Users'

    def __str__(self):
        return self.email

# settings/base.py
AUTH_USER_MODEL = 'users.User'
```

### 비밀번호 해싱 알고리즘
Django는 기본으로 PBKDF2를 사용합니다. 더 강력한 보안이 필요한 경우 Argon2 등을 추가할 수 있습니다.

```python
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher', # 강력 추천 (argon2-cffi 패키지 필요)
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
]
```

### 세션 관리

```python
# 세션 엔진 설정 (캐시 사용 권장)
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
SESSION_COOKIE_AGE = 3600 * 24 * 7  # 1주일
SESSION_SAVE_EVERY_REQUEST = False
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
```

## 인가 (Authorization)

### 권한 설정 및 검증

```python
# models.py
class Post(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(User, on_delete=models.CASCADE)

    class Meta:
        permissions = [
            ('can_publish', '포스트 발행 권한'),
            ('can_edit_others', '타인의 포스트 수정 권한'),
        ]

    def user_can_edit(self, user):
        """특정 사용자가 이 포스트를 수정할 수 있는지 확인."""
        return self.author == user or user.has_perm('app.can_edit_others')

# views.py
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin
from django.views.generic import UpdateView

class PostUpdateView(LoginRequiredMixin, PermissionRequiredMixin, UpdateView):
    model = Post
    permission_required = 'app.can_edit_others'
    raise_exception = True # 권한 없으면 리다이렉트 대신 403 에러 발생

    def get_queryset(self):
        """사용자가 자신이 작성한 글만 수정할 수 있도록 쿼리셋 제한."""
        return Post.objects.filter(author=self.request.user)
```

### DRF 커스텀 권한 클래스 (Permissions)

```python
# permissions.py
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """객체 소유자만 수정 가능, 나머지는 읽기만 가능."""
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS: # GET, HEAD, OPTIONS
            return True
        return obj.author == request.user

class IsAdminOrReadOnly(permissions.BasePermission):
    """관리자는 모든 작업 가능, 나머지는 읽기만 가능."""
    def has_permission(self, request, view):
        if request.method in permissions.SAFE_METHODS:
            return True
        return request.user and request.user.is_staff
```

## SQL 인젝션 방지

### Django ORM 활용 (매우 안전)

```python
# ✅ 좋음: Django ORM은 파라미터를 자동으로 이스케이프함
def get_user(username):
    return User.objects.get(username=username)

# ✅ 좋음: raw 쿼리 사용 시 파라미터 리스트 활용
def search_users(query):
    return User.objects.raw('SELECT * FROM users WHERE username = %s', [query])

# ❌ 나쁨: 사용자 입력을 문자열 포맷팅으로 직접 삽입 (매우 위험!)
def get_user_bad(username):
    return User.objects.raw(f"SELECT * FROM users WHERE username = '{username}'") # SQL 인젝션 취약!
```

## XSS (Cross-Site Scripting) 방지

### 템플릿 엔진의 자동 이스케이프

```django
{# Django는 기본적으로 변수를 자동 이스케이프함 - 안전 #}
{{ user_input }}

{# 신뢰할 수 있는 콘텐츠에만 제한적으로 safe 필터 사용 #}
{{ trusted_html|safe }}

{# 자바스크립트 내에 값을 넣을 때는 escapejs 사용 필수 #}
<script>
    var username = {{ username|escapejs }};
</script>
```

### 코드 내 안전한 문자열 처리

```python
from django.utils.html import format_html, escape
from django.utils.safestring import mark_safe

# ✅ 좋음: format_html을 사용하여 변수만 이스케이프 처리
def greet_user(username):
    return format_html('<span class="user">{}</span>', username)

# ❌ 나쁨: 검증되지 않은 사용자 입력을 mark_safe 처리 (위험!)
def render_bad(user_input):
    return mark_safe(user_input)
```

## CSRF (Cross-Site Request Forgery) 방지

### 기본 설정 적용

```python
# settings.py - CSRF는 기본 활성화되어 있음
CSRF_COOKIE_SECURE = True
CSRF_COOKIE_HTTPONLY = True
CSRF_COOKIE_SAMESITE = 'Lax'
CSRF_TRUSTED_ORIGINS = ['https://example.com']

# 템플릿 사용
<form method="post">
    {% csrf_token %}
    ...
</form>

# AJAX 요청 (헤더에 토큰 포함)
fetch('/api/endpoint/', {
    method: 'POST',
    headers: {
        'X-CSRFToken': getCookie('csrftoken'),
        'Content-Type': 'application/json',
    },
    body: JSON.stringify(data)
});
```

## 파일 업로드 보안

### 파일 검증 로직

```python
import os
from django.core.exceptions import ValidationError

def validate_file_extension(value):
    """허용된 확장자만 업로드 가능하도록 검증."""
    ext = os.path.splitext(value.name)[1]
    valid_extensions = ['.jpg', '.jpeg', '.png', '.pdf']
    if not ext.lower() in valid_extensions:
        raise ValidationError('지원하지 않는 파일 확장자입니다.')

def validate_file_size(value):
    """파일 크기 제한 (예: 최대 5MB)."""
    if value.size > 5 * 1024 * 1024:
        raise ValidationError('파일이 너무 큽니다. 최대 크기는 5MB입니다.')

# models.py
class Document(models.Model):
    file = models.FileField(
        upload_to='documents/%Y/%m/%d/', # 날짜별 경로 분리
        validators=[validate_file_extension, validate_file_size]
    )
```

## API 보안 및 속도 제한 (Rate Limiting)

```python
# settings.py (Django REST Framework)
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day', # 익명 사용자 제한
        'user': '1000/day', # 인증된 사용자 제한
        'upload': '10/hour', # 특정 기능 기능별 제한
    }
}
```

## 보안 이벤트 로깅

```python
LOGGING = {
    'version': 1,
    'loggers': {
        'django.security': { # 보안 관련 이벤트 로거
            'handlers': ['file'],
            'level': 'WARNING',
            'propagate': True,
        },
    },
}
```

## 빠른 보안 체크리스트

| 항목 | 설명 |
|-------|-------------|
| **DEBUG = False** | 운영 환경에서는 반드시 False로 설정 |
| **HTTPS 적용** | 모든 트래픽에 SSL 강제 적용 및 Secure 쿠키 사용 |
| **시크릿 키 보호** | 시크릿 키는 절대 코드에 포함하지 말고 환경 변수로 관리 |
| **비밀번호 검증** | 강력한 비밀번호 규칙 강제 적용 |
| **CSRF 보호** | 활성화 상태 확인 및 AJAX 요청 시 토큰 처리 확인 |
| **XSS 방지** | 템플릿의 자동 이스케이프를 신뢰하고 `|safe` 사용 최소화 |
| **SQL 인젝션** | 항상 ORM을 사용하고 직접적인 쿼리 문자열 조작 금지 |
| **파일 업로드** | 확장자 및 크기 검증 필수 |
| **API 제한** | 비정상적인 접근 방지를 위한 Throttling 설정 |
| **보안 헤더** | CSP, HSTS, X-Frame-Options 등 활성화 |
| **최신 버전 유지** | Django 및 모든 라이브러리를 최신 보안 패치 버전으로 유지 |

**기억하십시오**: 보안은 한 번의 설정으로 끝나는 것이 아니라 지속적으로 관리해야 하는 과정입니다. 주기적으로 코드를 진단하고 최신 보안 동향을 확인하십시오.
