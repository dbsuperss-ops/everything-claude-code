---
name: django-tdd
description: pytest-django를 이용한 Django 테스트 전략, TDD 방법론, factory_boy, 모킹(Mocking), 커버리지 측정 및 Django REST Framework API 테스트 가이드입니다.
origin: ECC
---

# TDD를 이용한 Django 테스트 (Django Testing with TDD)

pytest, factory_boy, Django REST Framework를 사용하는 Django 애플리케이션용 테스트 주도 개발(TDD) 가이드입니다.

## 활성화 시점

- 새로운 Django 애플리케이션 작성 시
- Django REST Framework(DRF) API 구현 시
- Django 모델, 뷰, 시리얼라이저 테스트 시
- Django 프로젝트를 위한 테스트 인프라 설정 시

## Django를 위한 TDD 워크플로우

### Red-Green-Refactor 사이클

1. **Red**: 실패하는 테스트를 먼저 작성합니다.
2. **Green**: 테스트를 통과시키기 위한 최소한의 코드를 작성합니다.
3. **Refactor**: 테스트 통과 상태를 유지하며 코드를 개선합니다.

## 설정 (Setup)

### pytest 설정

`pytest.ini` 파일을 통해 테스트 환경을 구성합니다. `--reuse-db` 및 `--nomigrations` 옵션을 사용하면 테스트 속도를 높일 수 있습니다.

### 테스트용 Settings

```python
# config/settings/test.py
from .base import *

# 테스트 속도를 위해 마이그레이션 비활성화 및 빠른 패스워드 해싱 사용
PASSWORD_HASHERS = ['django.contrib.auth.hashers.MD5PasswordHasher']
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

### conftest.py (Fixtures)

공통적으로 사용되는 유저 생성, 인증된 클라이언트 등을 픽스처(Fixture)로 정의합니다.

## Factory Boy

`factory_boy`를 사용하면 복잡한 모델 인스턴스를 간편하게 생성할 수 있습니다.

```python
# tests/factories.py
class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = User
    email = factory.Sequence(lambda n: f"user{n}@example.com")
    # ...
```

## 모델 테스트 (Model Testing)

모델의 생성, 서술형 표현(`__str__`), 커스텀 매니저 메서드, 비즈니스 로직(예: 재고 감소) 등을 테스트합니다.

## 뷰 테스트 (View Testing)

- **일반 뷰**: 페이지 접근성(200 OK), 컨텍스트 데이터, 템플릿 사용 여부, 로그인 리다이렉션 등을 테스트합니다.
- **API (DRF)**: 시리얼라이저 데이터 변환, 유효성 검사, 엔드포인트 응답(201 Created, 401 Unauthorized 등), 필터링 및 검색 기능을 테스트합니다.

## 모킹(Mocking) 및 패칭(Patching)

Stripe와 같은 외부 서비스 결제 연동이나 이메일 발송 등은 실제로 수행하지 않고 `unittest.mock`을 사용하여 성공/실패 시나리오를 시뮬레이션합니다.

## 통합 테스트 (Integration Testing)

회원가입 -> 로그인 -> 제품 탐색 -> 장바구니 추가 -> 결제 완료로 이어지는 전체 사용자 흐름(Flow)을 테스트합니다.

## 테스트 최선 관행

### 권장 사항 (DO)
- **팩토리(Factories) 사용**: 직접 객체를 생성하는 대신 팩토리를 활용하십시오.
- **하나의 테스트당 하나의 단언(Assertion)**: 테스트의 목적을 명확히 하십시오.
- **서술적인 테스트 이름**: `test_user_cannot_delete_others_post`와 같이 직관적으로 지으십시오.
- **경계값 테스트**: 빈 입력, None 값, 최대/최소값 등을 테스트하십시오.
- **픽스처(Fixtures) 활용**: 중복 코드를 제거하십시오.

### 지양 사항 (DON'T)
- **Django 내부 기능 테스트**: Django 프레임워크 자체의 동작은 신뢰하십시오.
- **상용 데이터베이스 사용**: 항상 테스트용 데이터베이스를 사용하십시오.
- **테스트 간 의존성**: 각 테스트는 어떤 순서로 실행되어도 성공해야 합니다.
- **프라이빗 메서드 테스트**: 공개 인터페이스(Public interface) 위주로 테스트하십시오.

## 커버리지 (Coverage)

`pytest-cov`를 사용하여 테스트가 전체 코드의 몇 %를 실행했는지 측정합니다. 일반적으로 **80% 이상의 전체 커버리지**를 목표로 하며, 핵심 비즈니스 로직인 모델과 서비스 레이어는 **90% 이상**을 권장합니다.

**기억하십시오**: 테스트는 문서입니다. 잘 작성된 테스트는 코드가 어떻게 작동해야 하는지 설명해줍니다. 테스트 코드를 단순하고 가독성 있으며 유지보수 가능하게 유지하십시오.
    
