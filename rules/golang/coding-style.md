---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go 코딩 스타일 (Go Coding Style)

> 이 파일은 [common/coding-style.md](../common/coding-style.md)을 Go 전용 내용으로 확장합니다.

## 포매팅

- **gofmt** 및 **goimports** 사용은 필수입니다 — 스타일 논쟁은 하지 않습니다.

## 설계 원칙

- 인터페이스를 받고, 구조체를 반환하십시오.
- 인터페이스를 작게 유지하십시오 (1~3개 메서드).

## 에러 처리

항상 맥락과 함께 에러를 래핑하십시오:

```go
if err != nil {
    return fmt.Errorf("유저 생성 실패: %w", err)
}
```

## 참조

포괄적인 Go 관용구 및 패턴에 대해서는 `golang-patterns` 스킬을 참조하십시오.
