---
name: django-security
description: Django 보안 최선 관행(Best practices), 인증(Authentication), 권한 부여(Authorization), CSRF 보호, SQL 인젝션 방지, XSS 방지 및 보안 배포 설정 가이드입니다.
origin: ECC
---

# Django 보안 최선 관행 (Django Security Best Practices)

공통적인 취약점으로부터 애플리케이션을 보호하기 위한 Django 보안 가이드라인을 안내합니다.

## 활성화 시점

- Django 인증 및 권한 부여 설정 시
- 사용자 권한 및 역할(Role) 구현 시
- 상용(Production) 환경 보안 설정 구성 시
- Django 애플리케이션 보안 취약점 검토 시
- 애플리케이션을 실제 서버에 배포할 때

## 핵심 보안 설정

### 상용 환경 설정 구성

```python
# settings/production.py
import os

DEBUG = False  # 중요: 상용 환경에서는 절대 True로 설정하지 마십시오.

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# 보안 헤더 설정
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000  # 1년
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY' # 클릭재킹 방지

# HTTPS 및 쿠키 설정
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
CSRF_COOKIE_SAMESITE = 'Lax'

# Secret key (반드시 환경 변수를 통해 설정)
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
if not SECRET_KEY:
    raise ImproperlyConfigured('DJANGO_SECRET_KEY 환경 변수가 필요합니다.')

# 비밀번호 유효성 검사 규칙
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator', 'OPTIONS': {'min_length': 12}},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]
```

## 인증 (Authentication)

### 커스텀 사용자 모델

```python
# apps/users/models.py
class User(AbstractUser):
    """보안 강화를 위한 커스텀 사용자 모델."""
    email = models.EmailField(unique=True)
    USERNAME_FIELD = 'email'  # 이메일을 사용자 아이디로 사용
    REQUIRED_FIELDS = ['username']

    class Meta:
        db_table = 'users'
```

### 비밀번호 해싱 (Password Hashing)

```python
# Django는 기본적으로 PBKDF2를 사용합니다. 보안을 더 강화하려면 Argon2 등을 추가하십시오.
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    # ...
]
```

## 권한 부여 (Authorization)

### 권한 및 믹스인 (Mixins)

```python
# views.py
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class PostUpdateView(LoginRequiredMixin, PermissionRequiredMixin, UpdateView):
    model = Post
    permission_required = 'app.can_edit_others'
    raise_exception = True  # 리다이렉트 대신 403 에러 반환
```

### DRF 커스텀 권한 클래스

```python
# permissions.py
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """객체 소유자만 수정 가능하도록 제한."""
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.author == request.user
```

## SQL 인젝션 방지

```python
# ✅ 좋음: Django ORM은 파라미터를 자동으로 이스케이프함
User.objects.get(username=username)

# ✅ 좋음: raw() 사용 시 파라미터 전달 방식 활용
User.objects.raw('SELECT * FROM users WHERE username = %s', [query])

# ❌ 나쁨: 사용자 입력을 직접 문자열에 포함하지 마십시오.
User.objects.raw(f'SELECT * FROM users WHERE username = {username}')  # 취약함!
```

## XSS (Cross-Site Scripting) 방지

```django
{# Django는 기본적으로 변수를 자동 이스케이프함 - 안전 #}
{{ user_input }}

{# 신뢰할 수 있는 콘텐츠에만 |safe를 사용하십시오 #}
{{ trusted_html|safe }}

{# JavaScript 내에서의 이스케이프 #}
<script>
    var username = {{ username|escapejs }};
</script>
```

## CSRF (Cross-Site Request Forgery) 보호

- `settings.py`에서 CSRF는 기본으로 활성화되어 있습니다.
- 템플릿의 form 태그 안에 `{% csrf_token %}`을 반드시 포함하십시오.
- AJAX 요청 시 헤더에 `X-CSRFToken`을 포함하십시오.

## 파일 업로드 보안

- 파일 확장자와 파일 크기를 검증하는 검증기(Validator)를 모델 필드에 연결하십시오.
- 실제 서비스 환경에서는 업로드된 파일을 static 파일 서버나 S3와 같은 별도 저장소에 보관하십시오.

## API 보안

- `REST_FRAMEWORK` 설정에서 `AnonRateThrottle` 및 `UserRateThrottle`을 사용하여 속도 제한(Rate Limiting)을 적용하십시오.
- 기본 인증 및 권한 클래스를 설정하여 모든 엔드포인트를 보호하십시오.

## 보안 체크리스트

| 항목 | 설명 |
|-------|-------------|
| `DEBUG = False` | 상용 환경에서는 절대 켜지 마십시오. |
| HTTPS 강제 | SSL 적용 및 쿠키 보안 설정을 확인하십시오. |
| 강력한 비밀키 | `SECRET_KEY`는 환경 변수로 관리하십시오. |
| 비밀번호 검증 | 모든 비밀번호 검증기를 활성화하십시오. |
| CSRF 보호 | 기본 설정을 건드리지 마십시오. |
| XSS 방지 | 사용자 입력에 `|safe` 필터를 남용하지 마십시오. |
| SQL 인젝션 | 항상 ORM을 사용하고 문자열 결합을 피하십시오. |
| 파일 업로드 | 형식과 크기를 반드시 검증하십시오. |
| 속도 제한 | API 엔드포인트에 쓰로틀링(Throttling)을 적용하십시오. |

**기억하십시오**: 보안은 제품이 아니라 지속적인 프로세스입니다. 정기적으로 보안 상태를 점검하고 업데이트하십시오.
    
