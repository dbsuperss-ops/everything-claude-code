---
description: "공통 규칙을 확장하는 Go 테스트"
globs: ["**/*.go", "**/go.mod", "**/go.sum"]
alwaysApply: false
---
# Go 테스트 (Testing)

> 이 문서는 공통 테스트 규칙을 기반으로 Go 언어에 특화된 내용을 확장합니다.

## 프레임워크

**테이블 기반 테스트(Table-driven tests)**와 함께 표준 `go test`를 사용하십시오.

## 경합 상태 감지 (Race Detection)

항상 `-race` 플래그를 포함하여 실행하십시오:

```bash
go test -race ./...
```

## 커버리지 (Coverage)

```bash
go test -cover ./...
```

## 참고 자료

상세한 Go 테스트 패턴 및 헬퍼 기능에 대해서는 `golang-testing` 스킬을 참조하십시오.
