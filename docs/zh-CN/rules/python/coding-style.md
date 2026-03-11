---
paths:
  - "**/*.py"
  - "**/*.pyi"
---

# Python 코딩 스타일 (Coding Style)

> 이 문서는 [common/coding-style.md](../common/coding-style.md)의 내용을 바탕으로 Python에 특화된 내용을 확장합니다.

## 표준 (Standards)

* **PEP 8** 명세 보급을 따릅니다.
* 모든 함수 시그니처에 **타입 힌트(Type Annotations)**를 사용하십시오.

## 불변성 (Immutability)

불변 데이터 구조 사용을 우선시하십시오:

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

## 포매팅 (Formatting)

* **black**을 사용하여 코드를 포매팅하십시오.
* **isort**를 사용하여 임포트(import) 순서를 정렬하십시오.
* **ruff**를 사용하여 코드 린팅(linting)을 수행하십시오.

## 참고 자료

Python의 종합적인 관례와 패턴에 대해서는 `python-patterns` 스킬을 참조하십시오.
