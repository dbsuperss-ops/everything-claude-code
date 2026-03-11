---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---

# Go 보안 (Security)

> 이 문서는 [common/security.md](../common/security.md)의 내용을 바탕으로 Go 언어에 특화된 내용을 확장합니다.

## 비밀 정보(Secrets) 관리

```go
apiKey := os.Getenv("OPENAI_API_KEY")
if apiKey == "" {
    log.Fatal("OPENAI_API_KEY가 설정되지 않았습니다")
}
```

## 보안 스캔

* **gosec**을 사용하여 정적 보안 분석을 수행하십시오:
  ```bash
  gosec ./...
  ```

## 컨텍스트 및 타임아웃

항상 `context.Context`를 사용하여 타임아웃을 제어하십시오:

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```
