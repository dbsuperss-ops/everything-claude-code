---
description: "공통 규칙을 확장하는 Go 후크(Hooks)"
globs: ["**/*.go", "**/go.mod", "**/go.sum"]
alwaysApply: false
---
# Go 후크 (Hooks)

> 이 문서는 공통 후크 규칙을 기반으로 Go 언어에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에서 다음 항목을 구성하십시오:

- **gofmt/goimports**: `.go` 파일 편집 후 자동 포매팅 수행
- **go vet**: `.go` 파일 편집 후 정적 분석 실행
- **staticcheck**: 수정된 패키지에 대해 확장 정적 검사 실행
