---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go 테스트 (Go Testing)

> 이 파일은 [common/testing.md](../common/testing.md)을 Go 전용 내용으로 확장합니다.

## 프레임워크

**테이블 기반 테스트(Table-driven tests)**와 함께 표준 `go test`를 사용하십시오.

## 레이스 감지 (Race Detection)

항상 `-race` 플래그와 함께 실행하십시오:

```bash
go test -race ./...
```

## 커버리지

```bash
go test -cover ./...
```

## 참조

상세한 Go 테스트 패턴 및 헬퍼에 대해서는 `golang-testing` 스킬을 참조하십시오.
