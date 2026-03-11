---
name: python-testing
description: pytest, TDD 방법론, 피스처(Fixture), 모킹(Mocking), 파라미터화 및 커버리지 요구사항을 포함한 Python 테스트 전략 가이드입니다.
origin: ECC
---

# Python 테스트 패턴 (Python Testing Patterns)

pytest와 TDD 방법론을 활용하여 신뢰할 수 있고 유지보수가 쉬운 Python 애플리케이션 테스트 작성 전략을 안내합니다.

## 활성화 시점

- 새로운 Python 코드를 작성할 때 (TDD: Red, Green, Refactor 원칙 준수)
- Python 프로젝트를 위한 테스트 스위트를 설계할 때
- 테스트 커버리지를 검토하고 높여야 할 때
- 테스트 환경 및 인프라를 설정할 때

## 핵심 테스트 철학

### 1. 테스트 주도 개발 (TDD)
- **RED**: 실패하는 테스트를 먼저 작성합니다.
- **GREEN**: 테스트를 통과시키기 위한 최소한의 코드를 작성합니다.
- **REFACTOR**: 테스트 통과 상태를 유지하며 코드를 개선합니다.

### 2. 커버리지 요구사항
- **목표**: 전체 코드 커버리지 80% 이상.
- **핵심 경로**: 비즈니스 로직 등 크리티컬한 경로는 100% 커버리지를 지향하십시오.
- `pytest --cov`를 사용하여 측정합니다.

## pytest 기본 및 주요 기능

### 1. 피스처 (Fixtures)
- 테스트 실행 전후의 데이터 준비와 정리를 담당합니다.
- `@pytest.fixture` 데코레이터를 사용하며, `yield`를 통해 셋업과 티어다운(Teardown)을 분리할 수 있습니다.
- `scope`(function, module, session 등)와 `autouse=True` 옵션을 활용하십시오.
- `conftest.py`를 통해 여러 테스트 파일에서 공유할 피스처를 정의하십시오.

### 2. 파라미터화 (Parametrization)
- `@pytest.mark.parametrize`를 사용하여 동일한 테스트 로직을 다양한 입력값과 기댓값으로 반복 실행할 수 있습니다.

### 3. 모킹 및 패칭 (Mocking & Patching)
- `unittest.mock`의 `patch`와 `Mock`을 사용하여 외부 API 호출, DB 연결 등을 대체하십시오.
- `side_effect`를 통해 예외 발생 상황을 시뮬레이션할 수 있습니다.

### 4. 비동기 테스트
- `pytest-asyncio`를 사용하고 `@pytest.mark.asyncio` 마커를 붙여 비동기 함수를 테스트하십시오.

### 5. 예외 테스트
- `with pytest.raises(ExceptionType):` 블록을 사용하여 특정 예외가 발생하는지 검증하십시오.

### 6. 테스트 구성 및 관리
- `tests/` 디렉토리 아래 `unit/`, `integration/`, `e2e/`로 구분하여 구성하십시오.
- 커스텀 마커(`@pytest.mark.slow` 등)를 정의하고 `pytest -m "not slow"`와 같이 선별적으로 실행하십시오.

## 최선 관행 (Best Practices)

### 권장 사항 (DO)
- **하나의 테스트는 하나의 동작**만 검증하십시오.
- **설명적인 이름**을 사용하십시오. (예: `test_user_login_fails_with_invalid_password`)
- 외부 서비스에 의존하지 않도록 **모킹**을 적극 활용하십시오.

### 지양 사항 (DON'T)
- 내부 구현(Internals)을 테스트하지 말고 **동작(Behavior)**을 테스트하십시오.
- 테스트 간에 **상태를 공유하지 마십시오**. 각 테스트는 독립적이어야 합니다.
- 테스트 코드에서 복잡한 조건문을 사용하지 마십시오.

**기억하십시오**: 테스트는 코드의 품질을 보증하는 것뿐만 아니라, 코드가 어떻게 동작해야 하는지를 보여주는 가장 좋은 문서입니다.
    
