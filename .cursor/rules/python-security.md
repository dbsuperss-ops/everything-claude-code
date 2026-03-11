---
description: "공통 규칙을 확장하는 Python 보안"
globs: ["**/*.py", "**/*.pyi"]
alwaysApply: false
---
# Python 보안 (Security)

> 이 문서는 공통 보안 규칙을 기반으로 Python에 특화된 내용을 확장합니다.

## 비밀 정보(Secret) 관리

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.environ["OPENAI_API_KEY"]  # 환경 변수가 없는 경우 KeyError 발생
```

## 보안 스캐닝

- 정적 보안 분석을 위해 **bandit**을 사용하십시오:
  ```bash
  bandit -r src/
  ```

## 참고 자료

Django를 사용하는 경우, Django에 특화된 보안 가이드라인을 위해 `django-security` 스킬을 참조하십시오.
