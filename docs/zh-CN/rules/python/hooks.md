---
paths:
  - "**/*.py"
  - "**/*.pyi"
---

# Python 후크 (Hooks)

> 이 문서는 [common/hooks.md](../common/hooks.md)의 내용을 바탕으로 Python에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에 다음 항목을 구성하십시오:

* **black/ruff**: `.py` 파일 편집 후 자동으로 포매팅을 수행합니다.
* **mypy/pyright**: `.py` 파일 편집 후 타입 체크를 수행합니다.

## 경고 사항

* 코드 내 `print()` 문 사용 시 경고를 표시합니다. (대신 `logging` 모듈을 사용해야 함)
