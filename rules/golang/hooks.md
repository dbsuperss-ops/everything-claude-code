---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go 훅 (Go Hooks)

> 이 파일은 [common/hooks.md](../common/hooks.md)을 Go 전용 내용으로 확장합니다.

## PostToolUse 훅

`~/.claude/settings.json`에서 설정하십시오:

- **gofmt/goimports**: `.go` 파일 수정 후 자동 포맷팅
- **go vet**: `.go` 파일 수정 후 정적 분석 실행
- **staticcheck**: 수정된 패키지에 대해 확장 정적 체크 실행
