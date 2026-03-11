---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python 패턴 (Python Patterns)

> 이 파일은 [common/patterns.md](../common/patterns.md)을 Python 전용 내용으로 확장합니다.

## 프로토콜 (Protocol, 덕 타이핑)

```python
from typing import Protocol

class Repository(Protocol):
    def find_by_id(self, id: str) -> dict | None: ...
    def save(self, entity: dict) -> dict: ...
```

## DTO로서의 데이터 클래스 (Dataclasses)

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
- 지연 평가(Lazy evaluation)와 메모리 효율적인 반복을 위해 제너레이터를 사용하십시오.

## 참조

데코레이터, 동시성, 패키지 구성 등을 포함한 포괄적인 패턴에 대해서는 `python-patterns` 스킬을 참조하십시오.
