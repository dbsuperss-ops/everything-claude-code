---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

# Swift 후크 (Hooks)

> 이 문서는 [common/hooks.md](../common/hooks.md)의 내용을 바탕으로 Swift에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에 다음 항목을 구성하십시오:

* **SwiftFormat**: `.swift` 파일 편집 후 자동으로 포매팅을 수행합니다.
* **SwiftLint**: `.swift` 파일 편집 후 코드 린팅을 수행합니다.
* **swift build**: 편집 후 수정된 패키지에 대해 타입 체크를 수행합니다.

## 경고 사항

* `print()` 문 사용을 경고합니다. 프로덕션 코드에서는 `os.Logger` 또는 구조화된 로깅(Structured logging)을 사용하십시오.
