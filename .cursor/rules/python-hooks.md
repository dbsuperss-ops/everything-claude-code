---
description: "공통 규칙을 확장하는 Python 후크(Hooks)"
globs: ["**/*.py", "**/*.pyi"]
alwaysApply: false
---
# Python 후크 (Hooks)

> 이 문서는 공통 후크 규칙을 기반으로 Python에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에서 다음 항목을 구성하십시오:

- **black/ruff**: `.py` 파일 편집 후 자동 포매팅 수행
- **mypy/pyright**: `.py` 파일 편집 후 타입 체크 실행

## 경고 사항 (Warnings)

- 편집된 파일에 `print()` 문이 있는 경우 경고합니다 (대신 `logging` 모듈을 사용하십시오).
