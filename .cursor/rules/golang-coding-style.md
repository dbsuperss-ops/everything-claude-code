---
description: "공통 규칙을 확장하는 Go 코딩 스타일"
globs: ["**/*.go", "**/go.mod", "**/go.sum"]
alwaysApply: false
---
# Go 코딩 스타일 (Coding Style)

> 이 문서는 공통 코딩 스타일 규칙을 기반으로 Go 언어에 특화된 내용을 확장합니다.

## 포매팅 (Formatting)

- **gofmt** 및 **goimports** 사용은 필수입니다. 스타일 논쟁은 무의미합니다.

## 설계 원칙

- 인터페이스를 입력으로 받고, 구조체를 반환하십시오. (Accept interfaces, return structs)
- 인터페이스는 작게 유지하십시오. (메서드 1~3개 권장)

## 에러 처리

항상 컨텍스트(Context)를 포함하여 에러를 래핑하십시오:

```go
if err != nil {
    return fmt.Errorf("사용자 생성 실패: %w", err)
}
```

## 참고 자료

Go 언어의 종합적인 관례와 패턴에 대해서는 `golang-patterns` 스킬을 참조하십시오.
