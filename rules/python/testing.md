---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python 테스트 (Python Testing)

> 이 파일은 [common/testing.md](../common/testing.md)을 Python 전용 내용으로 확장합니다.

## 프레임워크

테스트 프레임워크로 **pytest**를 사용하십시오.

## 커버리지

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

## 참조

상세한 pytest 패턴 및 피스처(Fixture)에 대해서는 `python-testing` 스킬을 참조하십시오.
