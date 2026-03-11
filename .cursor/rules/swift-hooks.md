---
description: "공통 규칙을 확장하는 Swift 후크(Hooks)"
globs: ["**/*.swift", "**/Package.swift"]
alwaysApply: false
---
# Swift 후크 (Hooks)

> 이 문서는 공통 후크 규칙을 기반으로 Swift에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에서 다음 항목을 구성하십시오:

- **SwiftFormat**: `.swift` 파일 편집 후 자동 포매팅 수행
- **SwiftLint**: `.swift` 파일 편집 후 린트(Lint) 검사 실행
- **swift build**: 편집 후 수정된 패키지에 대해 타입 체크 실행

## 경고 사항 (Warning)

- `print()` 문이 있는 경우 플래그를 표시합니다. 프로덕션 코드에서는 대신 `os.Logger` 또는 구조화된 로깅을 사용하십시오.
