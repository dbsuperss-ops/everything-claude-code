---
description: PEP 8 표준 준수, 타입 힌트, 보안 및 Pythonic 관용구 사용을 포함한 통합적인 Python 코드 리뷰를 수행합니다. python-reviewer 에이전트를 호출합니다.
---

# Python 코드 리뷰 (Code Review)

이 명령어는 **python-reviewer** 에이전트를 호출하여 Python 언어 전용의 심층적인 코드 리뷰를 수행합니다.

## 주요 기능

1. **Python 변경 사항 식별**: `git diff`를 통해 수정된 `.py` 파일을 찾습니다.
2. **정적 분석 실행**: `ruff`, `mypy`, `pylint`, `black --check` 등을 가동합니다.
3. **보안 스캔**: SQL 인젝션, 명령 인젝션, 안전하지 않은 역직렬화(Deserialization) 등을 점검합니다.
4. **타입 안전성 검토**: 타입 힌트(Type Hints) 누락 및 `mypy` 에러를 분석합니다.
5. **Pythonic 코드 확인**: PEP 8 표준 및 Python 특유의 최선 관행(Best Practices)을 준수하는지 검증합니다.
6. **보고서 생성**: 발견된 이슈를 심각도별로 분류하여 리포트를 작성합니다.

## 적용 시점

`/python-review` 명령어를 사용하는 경우:

* Python 코드를 작성하거나 수정한 직후
* 변경 사항을 커밋하기 전 최종 점검 시
* Python 코드가 포함된 풀 리퀘스트(PR)를 검토할 때
* 새로운 Python 코드베이스를 처음 분석할 때
* Pythonic한 코딩 패턴을 배우고 싶을 때

## 리뷰 카테고리 (심각도)

### 🔴 치명적 (필수 수정)
* SQL / 명령 인젝션 취약점
* 위험한 `eval()`, `exec()` 사용
* `pickle`을 이용한 안전하지 않은 역직렬화
* 하드코딩된 자격 증명 (API 키 등)
* YAML의 안전하지 않은 로드 (`yaml.load` 대신 `safe_load` 사용 권장)
* 에러를 숨기는 빈 `except:` 블록 (Bare except)

### 🟡 높음 (수정 권장)
* 공개 함수(Public functions)의 타입 힌트 누락
* 가변(Mutable) 객체를 기본 인자로 사용 (예: `def func(items=[])`)
* 예외를 조용히 무시함 (Silent exception swallowing)
* 리소스(파일, 네트워크 등) 사용 시 컨텍스트 매니저(`with`) 미사용
* 리스트 컴프리헨션 대신 C 스타일 루프 사용
* `isinstance()` 대신 `type()` 사용

### 🔵 중간 (고려 사항)
* PEP 8 스타일 가이드 위반
* 공개 함수의 Docstring 누락
* `logging` 대신 `print()` 사용
* 비효율적인 문자열 결합
* 매직 넘버(Magic numbers) 사용
* f-string 대신 오래된 방식의 문자열 포매팅 사용

---

## 실행되는 자동화 도구

```bash
# 타입 체크
mypy .

# 린트 및 포맷팅 확인
ruff check .
black --check .
isort --check-only .

# 보안 스캔
bandit -r .

# 의존성 감사
pip-audit
safety check

# 테스트 및 커버리지
pytest --cov=app --cov-report=term-missing
```

---

## 실행 예시

````text
사용자: /python-review

에이전트:
# Python 코드 리뷰 결과 보고서

## 리뷰 대상 파일
- app/routes/user.py (수정됨)
- app/services/auth.py (수정됨)

## 발견된 주요 이슈

[🔴 치명적] SQL 인젝션 취약점
파일: app/routes/user.py:42
내용: 사용자 입력이 SQL 쿼리에 직접 삽입되고 있습니다.
```python
query = f"SELECT * FROM users WHERE id = {user_id}"  # 위험!
```
해결책: 파라미터화된 쿼리를 사용하십시오.

[🟡 높음] 가변 기본 인자 사용
파일: app/services/auth.py:18
내용: 리스트를 기본 인자로 사용하여 상태가 공유될 위험이 있습니다.
```python
def process_items(items=[]):  # 위험!
```
해결책: 기본값을 `None`으로 설정하고 함수 내부에서 초기화하십시오.

## 최종 요약
* 치명적: 1 | 높음: 1 | 중간: 2
권고: ❌ 치명적인 이슈가 해결될 때까지 작업을 중단하십시오.
````

---

## 관련 정보

* **호출 에이전트**: `agents/python-reviewer.md`
* **관련 스킬**: `skills/python-patterns/`, `skills/python-testing/`
* **함께 쓰면 좋은 명령어**: `/tdd` (테스트 주도 구현), `/build-fix` (도구 실행 에러 시)

---

## 자주 발생하는 문제의 해결 방법 (Pythonic Fixes)

### 타입 힌트 추가
```python
# 수정 전
def calculate(x, y): return x + y

# 수정 후
from typing import Union
def calculate(x: Union[int, float], y: Union[int, float]) -> Union[int, float]:
    return x + y
```

### 컨텍스트 매니저 사용
```python
# 수정 전
f = open("file.txt"); data = f.read(); f.close()

# 수정 후
with open("file.txt") as f:
    data = f.read()
```

### 가변 기본 인자 수정
```python
# 수정 전
def append(value, items=[]): ...

# 수정 후
def append(value, items=None):
    if items is None:
        items = []
    ...
```

**핵심**: Python 코드 리뷰는 단순히 문법을 맞추는 것이 아니라, Python의 철학에 부합하고(Pythonic) 안전하며 성능이 우수한 코드를 작성하도록 돕는 과정입니다.
