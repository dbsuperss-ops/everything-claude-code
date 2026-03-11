---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---

# Go 테스트 (Testing)

> 이 문서는 [common/testing.md](../common/testing.md)의 내용을 바탕으로 Go 언어에 특화된 내용을 확장합니다.

## 프레임워크

표준 `go test` 패키지를 사용하며, **테이블 구동 테스트(Table-driven tests)** 방식을 채택하십시오.

## 레이스(Race) 감지

항상 `-race` 플래그를 사용하여 테스트를 실행하십시오:

```bash
go test -race ./...
```

## 커버리지

```bash
go test -cover ./...
```

## 참고 자료

상세한 Go 테스트 패턴과 보조 도구들에 대해서는 `golang-testing` 스킬을 참조하십시오.
