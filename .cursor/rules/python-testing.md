---
description: "공통 규칙을 확장하는 Python 테스트"
globs: ["**/*.py", "**/*.pyi"]
alwaysApply: false
---
# Python 테스트 (Testing)

> 이 문서는 공통 테스트 규칙을 기반으로 Python에 특화된 내용을 확장합니다.

## 프레임워크

테스트 프레임워크로 **pytest**를 사용하십시오.

## 커버리지 (Coverage)

```bash
pytest --cov=src --cov-report=term-missing
```

## 테스트 구성

테스트 분류를 위해 `pytest.mark`를 사용하십시오:

```python
import pytest

@pytest.mark.unit
def test_calculate_total():
    ...

@pytest.mark.integration
def test_database_connection():
    ...
```

## 참고 자료

상세한 pytest 패턴 및 픽스처(Fixtures)에 대해서는 `python-testing` 스킬을 참조하십시오.
