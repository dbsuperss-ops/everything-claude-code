---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go 보안 (Go Security)

> 이 파일은 [common/security.md](../common/security.md)을 Go 전용 내용으로 확장합니다.

## 비밀 정보 관리

```go
apiKey := os.Getenv("OPENAI_API_KEY")
if apiKey == "" {
    log.Fatal("OPENAI_API_KEY가 설정되지 않았습니다.")
}
```

## 보안 스캐닝

- 정적 보안 분석을 위해 **gosec**을 사용하십시오:
  ```bash
  gosec ./...
  ```

## 컨텍스트 및 타임아웃

타임아웃 제어를 위해 항상 `context.Context`를 사용하십시오:

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```
