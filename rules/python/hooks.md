---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python 훅 (Python Hooks)

> 이 파일은 [common/hooks.md](../common/hooks.md)을 Python 전용 내용으로 확장합니다.

## PostToolUse 훅

`~/.claude/settings.json`에서 설정하십시오:

- **black/ruff**: `.py` 파일 수정 후 자동 포맷팅
- **mypy/pyright**: `.py` 파일 수정 후 타입 체크 실행

## 경고

- 수정된 파일에 `print()` 문이 있는지 경고하십시오. (대신 `logging` 모듈 사용)
