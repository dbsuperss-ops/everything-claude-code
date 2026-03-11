---
paths:
  - "**/*.py"
  - "**/*.pyi"
---

# Python 보안 (Security)

> 이 문서는 [공통 보안 가이드](../common/security.md)의 내용을 바탕으로 Python에 특화된 내용을 확장합니다.

## 비밀 정보(Secrets) 관리

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.environ["OPENAI_API_KEY"]  # 설정되지 않은 경우 KeyError 발생
```

## 보안 스캔

* **bandit**을 사용하여 정적 보안 분석을 수행하십시오:
  ```bash
  bandit -r src/
  ```

## 참고 자료

Django를 사용하는 경우, `django-security` 스킬에서 특화된 보안 가이드를 확인하십시오.
