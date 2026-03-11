---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python 코딩 스타일 (Python Coding Style)

> 이 파일은 [common/coding-style.md](../common/coding-style.md)을 Python 전용 내용으로 확장합니다.

## 표준

- **PEP 8** 컨벤션을 따르십시오.
- 모든 함수 시그니처에 **타입 어노테이션(Type annotations)**을 사용하십시오.

## 불변성

불변 데이터 구조를 선호하십시오:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    name: str
    email: str

from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float
```

## 포매팅

- 코드 포매팅을 위해 **black** 사용
- 임포트 정렬을 위해 **isort** 사용
- 린팅을 위해 **ruff** 사용

## 참조

포괄적인 Python 관용구 및 패턴에 대해서는 `python-patterns` 스킬을 참조하십시오.
