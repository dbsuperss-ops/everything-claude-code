---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---

# Go 후크 (Hooks)

> 이 문서는 [common/hooks.md](../common/hooks.md)의 내용을 바탕으로 Go 언어에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에 다음 항목을 구성하십시오:

* **gofmt/goimports**: `.go` 파일 편집 후 자동으로 포매팅을 수행합니다.
* **go vet**: `.go` 파일 편집 후 정적 분석을 수행합니다.
* **staticcheck**: 수정된 패키지에 대해 확장 정적 검사를 수행합니다.
