---
이름: python-review
설명: PEP 8 준수 여부, 타입 힌트, 보안 및 파이썬스러운(Pythonic) 관용구에 대해 종합적인 Python 코드 리뷰를 수행합니다. python-reviewer 에이전트를 호출합니다.
---

# Python 코드 리뷰 (Python Code Review)

이 명령어는 종합적인 Python 전용 코드 리뷰를 위해 **python-reviewer** 에이전트를 호출합니다.

## 주요 기능

1. **Python 변경 사항 식별**: `git diff`를 통해 수정된 `.py` 파일을 찾습니다.
2. **정적 분석 실행**: `ruff`, `mypy`, `pylint`, `black --check` 등을 실행합니다.
3. **보안 스캔**: SQL 인젝션, 명령 인젝션, 안전하지 않은 역직렬화 등을 점검합니다.
4. **타입 안전성 리뷰**: 타입 힌트와 mypy 에러를 분석합니다.
5. **파이썬스러운 코드 확인**: 코드가 PEP 8 및 Python 최선 관행을 따르는지 확인합니다.
6. **보고서 생성**: 이슈를 심각도별로 분류하여 보고합니다.

## 사용 시점

다음과 같은 경우 `/python-review`를 사용하십시오:
- Python 코드를 작성하거나 수정했을 때
- Python 변경 사항을 커밋하기 전
- Python 코드가 포함된 Pull Request를 리뷰할 때
- 새로운 Python 코드베이스에 온보딩할 때
- 파이썬스러운 패턴과 관용구를 학습하고 싶을 때

## 리뷰 카테고리

### 치명적 (CRITICAL - 반드시 수정)
- SQL/명령 인젝션 취약점
- 안전하지 않은 eval/exec 사용
- Pickle을 사용한 안전하지 않은 역직렬화
- 하드코딩된 자격 증명 (API 키 등)
- YAML 안전하지 않은 로드(unsafe_load)
- 에러를 숨기는 빈 except 구문 (bare except)

### 높음 (HIGH - 수정 권장)
- 퍼블릭 함수에 타입 힌트 누락
- 가변 기본 인자 (Mutable default arguments)
- 예외를 조용히 무시함 (Swallowing exceptions)
- 리소스 관리에 컨텍스트 매니저(with 구문) 미사용
- 컴프리헨션 대신 C-스타일의 루프 사용
- isinstance() 대신 type() 사용
- Lock 없는 경쟁 상태(Race condition)

### 보통 (MEDIUM - 고려 사항)
- PEP 8 포맷 규칙 위반
- 퍼블릭 함수에 독스트링(docstring) 누락
- 로깅 대신 print 문 사용
- 비효율적인 문자열 연산
- 명명된 상수 없는 매직 넘버(Magic numbers)
- f-string 대신 이전 방식의 포맷팅 사용
- 불필요한 리스트 생성

## 자동 검사 도구 실행

```bash
# 타입 체크
mypy .

# 린트 및 포맷팅
ruff check .
black --check .
isort --check-only .

# 보안 스캔
bandit -r .

# 의존성 감사
pip-audit
safety check

# 테스트
pytest --cov=app --cov-report=term-missing
```

## 사용 예시

```text
사용자: /python-review

에이전트:
# Python 코드 리뷰 보고서

## 리뷰된 파일
- app/routes/user.py (수정됨)
- app/services/auth.py (수정됨)

## 정적 분석 결과
✓ ruff: 이슈 없음
✓ mypy: 에러 없음
⚠️ black: 2개 파일의 포맷팅 수정 필요
✓ bandit: 보안 이슈 없음

## 이슈 발견 사항

[치명적 - CRITICAL] SQL 인젝션 취약점
파일: app/routes/user.py:42
이슈: 사용자 입력이 SQL 쿼리에 직접 삽입됨
```python
query = f"SELECT * FROM users WHERE id = {user_id}"  # 나쁜 예시
```
수정 제안: 파라미터화된 쿼리 사용
```python
query = "SELECT * FROM users WHERE id = %s"  # 좋은 예시
cursor.execute(query, (user_id,))
```

[높음 - HIGH] 가변 기본 인자 (Mutable default argument)
파일: app/services/auth.py:18
이슈: 가변 기본 인자로 인해 상태가 공유됨
```python
def process_items(items=[]):  # 나쁜 예시
    items.append("new")
    return items
```
수정 제안: 기본값으로 None 사용
```python
def process_items(items=None):  # 좋은 예시
    if items is None:
        items = []
    items.append("new")
    return items
```

[보통 - MEDIUM] 타입 힌트 누락
파일: app/services/auth.py:25
이슈: 퍼블릭 함수에 타입 어노테이션이 없음
```python
def get_user(user_id):  # 나쁜 예시
    return db.find(user_id)
```
수정 제안: 타입 힌트 추가
```python
def get_user(user_id: str) -> Optional[User]:  # 좋은 예시
    return db.find(user_id)
```

## 요약
- 치명적(CRITICAL): 1
- 높음(HIGH): 1
- 보통(MEDIUM): 2

판정: ❌ 치명적인 이슈가 수정될 때까지 머지를 차단하십시오.

## 포맷팅 수정 필요
실행: `black app/routes/user.py app/services/auth.py`
```

## 승인 기준

| 상태 | 조건 |
|--------|-----------|
| ✅ 승인(Approve) | 치명적(CRITICAL) 또는 높음(HIGH) 이슈 없음 |
| ⚠️ 경고(Warning) | 보통(MEDIUM) 이슈만 발견됨 (주의하여 머지) |
| ❌ 차단(Block) | 치명적(CRITICAL) 또는 높음(HIGH) 이슈 발견됨 |

## 다른 명령어와의 통합

- 테스트 통과 여부 확인을 위해 `/tdd` 먼저 실행
- Python 전용 사항이 아닌 일반적인 사항은 `/code-review` 활용
- 커밋 전 반드시 `/python-review` 실행
- 정적 분석 도구 자체에서 에러 발생 시 `/build-fix` 사용

## 프레임워크별 리뷰 항목

### Django 프로젝트
리뷰어 확인 사항:
- N+1 쿼리 이슈 (`select_related`, `prefetch_related` 사용 여부)
- 모델 변경에 따른 마이그레이션 파일 누락 여부
- ORM으로 처리 가능한 곳에 생(raw) SQL 사용 여부
- 다단계 작업 시 `transaction.atomic()` 누락 여부

### FastAPI 프로젝트
리뷰어 확인 사항:
- CORS 설정 오류
- 요청 검증을 위한 Pydantic 모델 사용 여부
- 응답 모델의 정확성
- 올바른 async/await 사용
- 의존성 주입(Dependency Injection) 패턴

### Flask 프로젝트
리뷰어 확인 사항:
- 컨텍스트 관리 (app context, request context)
- 적절한 에러 처리
- Blueprint를 활용한 조직화
- 설정값 관리 방식

## 관련 정보

- 에이전트: `agents/python-reviewer.md`
- 스킬: `skills/python-patterns/`, `skills/python-testing/`

## 일반적인 수정 사례 (Common Fixes)

### 타입 힌트 추가
```python
# 수정 전
def calculate(x, y):
    return x + y

# 수정 후
from typing import Union

def calculate(x: Union[int, float], y: Union[int, float]) -> Union[int, float]:
    return x + y
```

### 컨텍스트 매니저 사용
```python
# 수정 전
f = open("file.txt")
data = f.read()
f.close()

# 수정 후
with open("file.txt") as f:
    data = f.read()
```

### 리스트 컴프리헨션 사용
```python
# 수정 전
result = []
for item in items:
    if item.active:
        result.append(item.name)

# 수정 후
result = [item.name for item in items if item.active]
```

### 가변 기본값 수정
```python
# 수정 전
def append(value, items=[]):
    items.append(value)
    return items

# 수정 후
def append(value, items=None):
    if items is None:
        items = []
    items.append(value)
    return items
```

### f-strings 사용 (Python 3.6+)
```python
# 수정 전
name = "Alice"
greeting = "Hello, " + name + "!"
greeting2 = "Hello, {}".format(name)

# 수정 후
greeting = f"Hello, {name}!"
```

### 루프 내 문자열 연결 수정
```python
# 수정 전
result = ""
for item in items:
    result += str(item)

# 수정 후
result = "".join(str(item) for item in items)
```

## Python 버전 호환성

리뷰어는 코드가 더 최신 Python 버전의 기능을 사용하는지 확인합니다:

| 기능 | 최소 Python 버전 |
|---------|----------------|
| 타입 힌트 | 3.5+ |
| f-strings | 3.6+ |
| 바다코끼리 연산자 (`:=`) | 3.8+ |
| 위치 전용 파라미터 (Position-only) | 3.8+ |
| Match 구문 | 3.10+ |
| 타입 유니온 (&#96;x &#124; None&#96;) | 3.10+ |

프로젝트의 `pyproject.toml` 또는 `setup.py`에 올바른 최소 Python 버전이 명시되어 있는지 확인하십시오.
