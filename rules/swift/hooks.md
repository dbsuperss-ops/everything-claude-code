---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Swift 훅 (Swift Hooks)

> 이 파일은 [common/hooks.md](../common/hooks.md)을 Swift 전용 내용으로 확장합니다.

## PostToolUse 훅

`~/.claude/settings.json`에서 설정하십시오:

- **SwiftFormat**: `.swift` 파일 수정 후 자동 포맷팅
- **SwiftLint**: `.swift` 파일 수정 후 린트 체크 실행
- **swift build**: 수정 후 해당 패키지에 대해 타입 체크 실행

## 경고

`print()` 문을 감시하십시오 — 프로덕션 코드에서는 대신 `os.Logger`나 구조적 로깅을 사용하십시오.
