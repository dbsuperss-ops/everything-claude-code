---
name: python-testing
description: pytest를 사용한 Python 테스트 전략 가이드입니다. TDD 방법론, 픽스처(Fixtures), 모킹(Mocking), 파라미터화(Parameterization) 및 커버리지 요구 사항을 다룹니다.
origin: ECC
---

# Python 테스트 패턴

pytest, TDD 방법론 및 베스트 프랙티스를 활용한 Python 애플리케이션의 종합적인 테스트 전략입니다.

## 적용 시점

* 새로운 Python 코드를 작성할 때 (TDD 준수: Red, Green, Refactor)
* Python 프로젝트를 위한 테스트 스위트를 설계할 때
* Python 테스트 커버리지를 검토할 때
* 테스트 인프라를 설정할 때

## 핵심 테스트 철학

### 테스트 주도 개발 (TDD)

항상 TDD 사이클을 따르십시오:

1. **Red**: 예상되는 동작에 대해 실패하는 테스트를 먼저 작성합니다.
2. **Green**: 테스트를 통과시키기 위한 최소한의 코드를 작성합니다.
3. **Refactor**: 테스트 통과를 유지하면서 코드를 개선하고 정리합니다.

```python
# 1단계: 실패하는 테스트 작성 (RED)
def test_add_numbers():
    result = add(2, 3)
    assert result == 5

# 2단계: 최소한의 구현 (GREEN)
def add(a, b):
    return a + b

# 3단계: 필요한 경우 리팩토링 (REFACTOR)
```

### 커버리지 요구 사항

* **목표**: 전체 코드 커버리지 80% 이상
* **핵심 경로**: 주요 비즈니스 로직은 100% 커버리지 필요
* `pytest --cov`를 사용하여 커버리지를 측정하십시오.

```bash
pytest --cov=mypackage --cov-report=term-missing --cov-report=html
```

## pytest 기초

### 기본 테스트 구조

```python
import pytest

def test_addition():
    """기본 덧셈 테스트"""
    assert 2 + 2 == 4

def test_string_uppercase():
    """문자열 대문자 변환 테스트"""
    text = "hello"
    assert text.upper() == "HELLO"

def test_list_append():
    """리스트 추가 테스트"""
    items = [1, 2, 3]
    items.append(4)
    assert 4 in items
    assert len(items) == 4
```

### 단언 (Assertions)

```python
# 값 일치 여부
assert result == expected

# 값 불일치 여부
assert result != unexpected

# 진리값 확인
assert result  # Truthy
assert not result  # Falsy
assert result is True  # 정확히 True
assert result is None  # 정확히 None

# 포함 여부
assert item in collection
assert item not in collection

# 비교 연산
assert result > 0
assert 0 <= result <= 100

# 타입 확인
assert isinstance(result, str)

# 예외 발생 테스트 (권장 방식)
with pytest.raises(ValueError):
    raise ValueError("에러 메시지")

# 예외 메시지 정규식 매칭 확인
with pytest.raises(ValueError, match="invalid input"):
    raise ValueError("invalid input provided")
```

## 픽스처 (Fixtures)

### 기본 픽스처 사용

```python
import pytest

@pytest.fixture
def sample_data():
    """샘플 데이터를 제공하는 픽스처"""
    return {"name": "Alice", "age": 30}

def test_sample_data(sample_data):
    """픽스처를 사용하는 테스트"""
    assert sample_data["name"] == "Alice"
    assert sample_data["age"] == 30
```

### 설정(Setup) 및 정리(Teardown)를 포함한 픽스처

```python
@pytest.fixture
def database():
    """설정 및 정리가 포함된 픽스처"""
    # 설정 (Setup)
    db = Database(":memory:")
    db.create_tables()
    yield db  # 테스트에 제공

    # 정리 (Teardown)
    db.close()
```

### 픽스처 범위 (Scopes)

* `function` (기본값): 각 테스트 메서드마다 실행
* `module`: 파이썬 모듈(파일)당 한 번 실행
* `class`: 클래스당 한 번 실행
* `session`: 전체 테스트 세션 동안 한 번 실행

### 자동 실행 픽스처 (autouse)

```python
@pytest.fixture(autouse=True)
def reset_config():
    """모든 테스트 전에 자동으로 실행됨"""
    Config.reset()
    yield
    Config.cleanup()
```

## 파라미터화 (Parameterization)

다양한 입력값에 대해 동일한 테스트를 반복 실행할 때 사용합니다.

```python
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("PyThOn", "PYTHON"),
])
def test_uppercase(input, expected):
    assert input.upper() == expected
```

## 모킹 (Mocking) 및 패치 (Patching)

외부 의존성(API, DB 등)을 가짜 객체로 대체합니다.

```python
from unittest.mock import patch, Mock

@patch("mypackage.external_api_call")
def test_with_mock(api_call_mock):
    """외부 API 호출을 모킹한 테스트"""
    api_call_mock.return_value = {"status": "success"}

    result = my_function()

    api_call_mock.assert_called_once()
    assert result["status"] == "success"
```

## 비동기 코드 테스트 (pytest-asyncio)

```python
import pytest

@pytest.mark.asyncio
async def test_async_function():
    """비동기 함수 테스트"""
    result = await async_add(2, 3)
    assert result == 5
```

## 베스트 프랙티스

### 권장 사항 (Dos)
* **TDD를 따르십시오**: 코드 구현 전에 테스트를 먼저 작성하십시오.
* **단일 책임**: 하나의 테스트는 하나의 동작만 검증해야 합니다.
* **설명적인 이름**: `test_user_login_with_invalid_password_fails` 처럼 의도가 드러나게 지으십시오.
* **픽스처 활용**: 반복되는 준비 과정은 픽스처로 추상화하십시오.
* **독립성 유지**: 테스트는 다른 테스트의 실행 결과에 의존해서는 안 됩니다.

### 지양 사항 (Donts)
* **구현 세부 사항 테스트 지양**: 내부 로직이 아닌 '행위'와 '결과'를 테스트하십시오.
* **복잡한 논리 금지**: 테스트 코드 자체에 복잡한 if/for문을 넣지 마십시오.
* **외부 서비스 의존 금지**: 실제 네트워크나 외부 API에 의존하지 말고 모킹하십시오.
* **공유 상태 주의**: 테스트 간에 데이터를 공유하여 부수 효과가 발생하지 않도록 하십시오.

## 테스트 실행 명령어

```bash
# 모든 테스트 실행
pytest

# 특정 파일 실행
pytest tests/test_utils.py

# 상세 결과 출력 (Verbose)
pytest -v

# 커버리지 보고서 생성
pytest --cov=mypackage --cov-report=html

# 실패한 테스트부터 재실행
pytest --lf

# 디버거와 함께 실행 (실패 지점에서 멈춤)
pytest --pdb
```

**핵심**: 테스트는 안전한 리팩토링과 빠른 개발을 가능하게 하는 보험입니다. 자동화된 테스트를 통해 코드의 품질과 신뢰성을 확보하십시오.
