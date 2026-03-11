---
description: "공통 규칙을 확장하는 Python 코딩 스타일"
globs: ["**/*.py", "**/*.pyi"]
alwaysApply: false
---
# Python 코딩 스타일 (Coding Style)

> 이 문서는 공통 코딩 스타일 규칙을 기반으로 Python에 특화된 내용을 확장합니다.

## 표준 (Standards)

- **PEP 8** 컨벤션을 따릅니다.
- 모든 함수 서명에 **타입 어노테이션(Type annotations)**을 사용하십시오.

## 불변성 (Immutability)

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

## 포매팅 (Formatting)

- 코드 포매팅을 위해 **black**을 사용하십시오.
- 임포트 정렬을 위해 **isort**를 사용하십시오.
- 린팅을 위해 **ruff**를 사용하십시오.

## 참고 자료

Python의 종합적인 관례와 패턴에 대해서는 `python-patterns` 스킬을 참조하십시오.
