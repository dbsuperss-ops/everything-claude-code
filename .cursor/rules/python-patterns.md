---
description: "공통 규칙을 확장하는 Python 패턴"
globs: ["**/*.py", "**/*.pyi"]
alwaysApply: false
---
# Python 패턴 (Patterns)

> 이 문서는 공통 패턴 규칙을 기반으로 Python에 특화된 내용을 확장합니다.

## 프로토콜 (Protocol, 덕 타이핑)

```python
from typing import Protocol

class Repository(Protocol):
    def find_by_id(self, id: str) -> dict | None: ...
    def save(self, entity: dict) -> dict: ...
```

## DTO로서의 데이터 클래스 (Dataclass)

```python
from dataclasses import dataclass

@dataclass
class CreateUserRequest:
    name: str
    email: str
    age: int | None = None
```

## 컨텍스트 매니저 및 제너레이터

- 리소스 관리를 위해 컨텍스트 매니저(`with` 문)를 사용하십시오.
- 지연 평가(Lazy evaluation)와 메모리 효율적인 반복 작업을 위해 제너레이터를 사용하십시오.

## 참고 자료

데코레이터, 동시성, 패키지 구성 등을 포함한 Python의 종합적인 패턴에 대해서는 `python-patterns` 스킬을 참조하십시오.
