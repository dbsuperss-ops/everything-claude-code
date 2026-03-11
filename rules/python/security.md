---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python 보안 (Python Security)

> 이 파일은 [common/security.md](../common/security.md)을 Python 전용 내용으로 확장합니다.

## 비밀 정보 관리

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.environ["OPENAI_API_KEY"]  # 설정되지 않은 경우 KeyError 발생
```

## 보안 스캐닝

- 정적 보안 분석을 위해 **bandit**을 사용하십시오:
  ```bash
  bandit -r src/
  ```

## 참조

Django를 사용하는 경우, Django 전용 보안 가이드라인에 대해서는 `django-security` 스킬을 참조하십시오.
