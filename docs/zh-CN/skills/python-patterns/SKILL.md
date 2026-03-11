---
name: python-patterns
description: 파이썬다운(Pythonic) 관용구, PEP 8 표준, 타입 힌트, 그리고 견고하고 효율적이며 유지보수가 용이한 Python 애플리케이션 구축을 위한 베스트 프랙티스입니다.
origin: ECC
---

# Python 개발 패턴

견고하고 효율적이며 유지보수가 용이한 애플리케이션을 구축하기 위한 Python 관용적 패턴과 베스트 프랙티스입니다.

## 적용 시점

* 새로운 Python 코드를 작성할 때
* Python 코드를 리뷰할 때
* 기존 Python 코드를 리팩토링할 때
* Python 패키지 또는 모듈을 설계할 때

## 핵심 원칙

### 1. 가독성이 중요하다 (Readability Matters)

Python은 가독성을 최우선으로 합니다. 코드는 명확하고 이해하기 쉬워야 합니다.

```python
# ✅ 좋은 예: 명확하고 읽기 쉬움
def get_active_users(users: list[User]) -> list[User]:
    """제공된 리스트에서 활성 사용자만 반환합니다."""
    return [user for user in users if user.is_active]

# ❌ 나쁜 예: 기교를 부렸으나 혼란스러움
def get_active_users(u):
    return [x for x in u if x.a]
```

### 2. 명시적인 것이 암시적인 것보다 낫다 (Explicit over Implicit)

마법 같은 코드(Magic)를 피하십시오. 코드가 무엇을 하는지 명확하게 기술해야 합니다.

```python
# ✅ 좋은 예: 명시적인 설정
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# ❌ 나쁜 예: 숨겨진 부수 효과 (Side effects)
import some_module
some_module.setup()  # 내부에서 무슨 일이 일어나는지 알기 어렵습니다.
```

### 3. EAFP - 허락보다 용서가 쉽다 (Easier to Ask Forgiveness than Permission)

Python은 조건을 미리 확인하는 것보다 예외 처리를 사용하는 방식을 선호합니다.

```python
# ✅ 좋은 예: EAFP 스타일
def get_value(dictionary: dict, key: str) -> Any:
    try:
        return dictionary[key]
    except KeyError:
        return default_value

# ❌ 나쁜 예: LBYL (Look Before You Leap) 스타일
def get_value(dictionary: dict, key: str) -> Any:
    if key in dictionary:
        return dictionary[key]
    else:
        return default_value
```

## 타입 힌트 (Type Hints)

### 기본 타입 어노테이션

```python
from typing import Optional, List, Dict, Any

def process_user(
    user_id: str,
    data: Dict[str, Any],
    active: bool = True
) -> Optional[User]:
    """사용자를 처리하고 업데이트된 User 객체 또는 None을 반환합니다."""
    if not active:
        return None
    return User(user_id, data)
```

### 현대적인 타입 힌트 (Python 3.9+)

```python
# Python 3.9+ - 내장 타입을 직접 사용하십시오.
def process_items(items: list[str]) -> dict[str, int]:
    return {item: len(item) for item in items}

# Python 3.8 이하 - typing 모듈을 사용하십시오.
from typing import List, Dict

def process_items(items: List[str]) -> Dict[str, int]:
    return {item: len(item) for item in items}
```

### 타입 별칭(Alias) 및 TypeVar

```python
from typing import TypeVar, Union

# 복잡한 타입을 위한 타입 별칭
JSON = Union[dict[str, Any], list[Any], str, int, float, bool, None]

def parse_json(data: str) -> JSON:
    return json.loads(data)

# 제네릭(Generic) 타입
T = TypeVar('T')

def first(items: list[T]) -> T | None:
    """첫 번째 항목을 반환하거나, 리스트가 비어있으면 None을 반환합니다."""
    return items[0] if items else None
```

### 프로토콜(Protocol) 기반 덕 타이핑

```python
from typing import Protocol

class Renderable(Protocol):
    def render(self) -> str:
        """객체를 문자열로 렌더링합니다."""

def render_all(items: list[Renderable]) -> str:
    """Renderable 프로토콜을 구현한 모든 항목을 렌더링합니다."""
    return "\n".join(item.render() for item in items)
```

## 에러 처리 패턴

### 구체적인 예외 처리

```python
# ✅ 좋은 예: 구체적인 예외 상황을 잡으십시오.
def load_config(path: str) -> Config:
    try:
        with open(path) as f:
            return Config.from_json(f.read())
    except FileNotFoundError as e:
        raise ConfigError(f"설정 파일을 찾을 수 없습니다: {path}") from e
    except json.JSONDecodeError as e:
        raise ConfigError(f"설정 파일의 JSON 형식이 잘못되었습니다: {path}") from e

# ❌ 나쁜 예: 광범위한 except 사용
def load_config(path: str) -> Config:
    try:
        with open(path) as f:
            return Config.from_json(f.read())
    except:
        return None  # 조용한 실패(Silent failure)!
```

### 예외 체이닝 (Exception Chaining)

```python
def process_data(data: str) -> Result:
    try:
        parsed = json.loads(data)
    except json.JSONDecodeError as e:
        # Traceback을 보존하기 위해 'from e'를 사용하여 예외를 연결하십시오.
        raise ValueError(f"데이터 파싱 실패: {data}") from e
```

### 사용자 정의 예외 계층 구조

```python
class AppError(Exception):
    """모든 애플리케이션 에러의 기본 예외 클래스입니다."""
    pass

class ValidationError(AppError):
    """입력 데이터 검증 실패 시 발생합니다."""
    pass

class NotFoundError(AppError):
    """요청한 리소스를 찾을 수 없을 때 발생합니다."""
    pass

# 사용 예시
def get_user(user_id: str) -> User:
    user = db.find_user(user_id)
    if not user:
        raise NotFoundError(f"사용자를 찾을 수 없습니다: {user_id}")
    return user
```

## 컨텍스트 매니저 (Context Managers)

### 리소스 관리

```python
# ✅ 좋은 예: context manager 활용
def process_file(path: str) -> str:
    with open(path, 'r') as f:
        return f.read()

# ❌ 나쁜 예: 수동 리소스 관리
def process_file(path: str) -> str:
    f = open(path, 'r')
    try:
        return f.read()
    finally:
        f.close()
```

### 커스텀 컨텍스트 매니저 (@contextmanager)

```python
from contextlib import contextmanager

@contextmanager
def timer(name: str):
    """코드 블록의 실행 시간을 측정하는 컨텍스트 매니저입니다."""
    start = time.perf_counter()
    yield
    elapsed = time.perf_counter() - start
    print(f"{name} 수행 시간: {elapsed:.4f}초")

# 사용 예시
with timer("데이터 처리"):
    process_large_dataset()
```

## 컴프리헨션 및 제너레이터

### 리스트 컴프리헨션 (List Comprehensions)

```python
# ✅ 좋은 예: 단순 변환 시 리스트 컴프리헨션 사용
names = [user.name for user in users if user.is_active]

# ❌ 나쁜 예: 수동 루프 사용
names = []
for user in users:
    if user.is_active:
        names.append(user.name)

# 컴프리헨션이 너무 복잡해지면 일반 함수나 루프로 분리하십시오.
# ❌ 너무 복잡한 예
result = [x * 2 for x in items if x > 0 if x % 2 == 0]

# ✅ 좋은 예: 제너레이터 함수 사용
def filter_and_transform(items: Iterable[int]) -> list[int]:
    result = []
    for x in items:
        if x > 0 and x % 2 == 0:
            result.append(x * 2)
    return result
```

### 제너레이터 표현식 (Generator Expressions)

```python
# ✅ 좋은 예: 지연 평가(Lazy evaluation)를 위한 제너레이터
total = sum(x * x for x in range(1_000_000))

# ❌ 나쁜 예: 메모리에 거대한 중간 리스트를 생성함
total = sum([x * x for x in range(1_000_000)])
```

### 제너레이터 함수

```python
def read_large_file(path: str) -> Iterator[str]:
    """대용량 파일을 한 줄씩 읽습니다."""
    with open(path) as f:
        for line in f:
            yield line.strip()

# 사용 예시
for line in read_large_file("huge.txt"):
    process(line)
```

## 데이터 클래스 및 네임드 튜플

### 데이터 클래스 (Dataclasses)

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class User:
    """__init__, __repr__, __eq__가 자동으로 생성되는 사용자 엔티티입니다."""
    id: str
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    is_active: bool = True

# 사용 예시
user = User(
    id="123",
    name="Alice",
    email="alice@example.com"
)
```

### 네임드 튜플 (NamedTuple)

```python
from typing import NamedTuple

class Point(NamedTuple):
    """불변(Immutable) 2D 좌표 점입니다."""
    x: float
    y: float

    def distance(self, other: 'Point') -> float:
        return ((self.x - other.x) ** 2 + (self.y - other.y) ** 2) ** 0.5
```

## 데코레이터 (Decorators)

### 함수형 데코레이터

```python
import functools
import time

def timer(func: Callable) -> Callable:
    """함수 실행 시간을 측정하는 데코레이터입니다."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} 수행 시간: {elapsed:.4f}초")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
```

## 동시성 패턴 (Concurrency)

### I/O 집약적 작업을 위한 스레딩

```python
import concurrent.futures

def fetch_url(url: str) -> str:
    """URL 데이터 가져오기 (I/O 작업)"""
    import urllib.request
    with urllib.request.urlopen(url) as response:
        return response.read().decode()

def fetch_all_urls(urls: list[str]) -> dict[str, str]:
    """스레드 풀을 사용하여 병렬로 여러 URL을 가져옵니다."""
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
        future_to_url = {executor.submit(fetch_url, url): url for url in urls}
        # ... 결과 수집 로직
```

### CPU 집약적 작업을 위한 멀티프로세싱

```python
def process_data(data: list[int]) -> int:
    """CPU를 많이 사용하는 연산"""
    return sum(x ** 2 for x in data)

def process_all(datasets: list[list[int]]) -> list[int]:
    """멀티프로세스를 사용하여 연산 효율을 높입니다."""
    with concurrent.futures.ProcessPoolExecutor() as executor:
        results = list(executor.map(process_data, datasets))
    return results
```

## 메모리 및 성능 최적화

### __slots__ 활용 (메모리 절약)

```python
# ❌ 일반 클래스는 __dict__를 사용하여 메모리를 더 많이 소모합니다.
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# ✅ __slots__는 속성 저장을 최적화하여 메모리 사용량을 줄입니다.
class Point:
    __slots__ = ['x', 'y']
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

### 루프 내 문자열 결합 지양

```python
# ❌ 나쁜 예: 문자열 불변성 때문에 O(n²) 성능 발생
result = ""
for item in items:
    result += str(item)

# ✅ 좋은 예: join을 사용하여 O(n)으로 처리
result = "".join(str(item) for item in items)
```

## 파이썬 도구 통합

기본 명령어 활용:
```bash
# 코드 포맷팅
black .
isort .

# 린팅 (Linting)
ruff check .

# 타입 체크
mypy .

# 테스트 및 커버리지
pytest --cov=mypackage --cov-report=html
```

## 피해야 할 안티 패턴

```python
# ❌ 가변 객체를 기본 인자로 사용하지 마십시오 (Mutable default arguments)
def append_to(item, items=[]):
    items.append(item)
    return items

# ✅ None을 사용하고 함수 내부에서 리스트를 생성하십시오.
def append_to(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# ❌ type()으로 타입 비교 지양 -> isinstance() 사용 권장
# ❌ value == None 지양 -> value is None 사용 권장
# ❌ from module import * 지양 -> 명시적 import 사용 권장
```

**핵심**: Python 코드는 가독성이 높고, 명시적이어야 하며, 놀라움 최소화의 원칙(Principle of least astonishment)을 따라야 합니다. 화려한 기교보다는 명확성을 우선시하십시오.
