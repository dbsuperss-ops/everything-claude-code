---
description: "공통 규칙을 확장하는 Go 보안"
globs: ["**/*.go", "**/go.mod", "**/go.sum"]
alwaysApply: false
---
# Go 보안 (Security)

> 이 문서는 공통 보안 규칙을 기반으로 Go 언어에 특화된 내용을 확장합니다.

## 비밀 정보(Secret) 관리

```go
apiKey := os.Getenv("OPENAI_API_KEY")
if apiKey == "" {
    log.Fatal("OPENAI_API_KEY가 구성되지 않았습니다")
}
```

## 보안 스캐닝

- 정적 보안 분석을 위해 **gosec**을 사용하십시오:
  ```bash
  gosec ./...
  ```

## 컨텍스트(Context) 및 타임아웃

제한 시간(Timeout) 제어를 위해 항상 `context.Context`를 사용하십시오:

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```
