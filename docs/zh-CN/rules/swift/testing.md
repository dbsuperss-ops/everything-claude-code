---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

# Swift 테스트 (Testing)

> 이 문서는 [common/testing.md](../common/testing.md)의 내용을 바탕으로 Swift에 특화된 내용을 확장합니다.

## 프레임워크

새로운 테스트 작성 시 **Swift Testing**(`import Testing`) 프레임워크를 사용하십시오. `@Test` 매크로와 `#expect` 구문을 활용합니다:

```swift
@Test("사용자 생성 시 이메일 유효성 검사")
func userCreationValidatesEmail() throws {
    #expect(throws: ValidationError.invalidEmail) {
        try User(email: "not-an-email")
    }
}
```

## 테스트 격리 (Isolation)

각 테스트는 독립된 인스턴스를 보장받습니다. `init`에서 설정(Set up)하고 `deinit`에서 해제(Teardown)하십시오. 테스트 간에 공유되는 가변 상태가 없어야 합니다.

## 파라미터화 테스트 (Parameterized Tests)

```swift
@Test("포맷 유효성 검사", arguments: ["json", "xml", "csv"])
func validatesFormat(format: String) throws {
    let parser = try Parser(format: format)
    #expect(parser.isValid)
}
```

## 커버리지

```bash
swift test --enable-code-coverage
```

## 참고 자료

프로토콜 기반의 의존성 주입과 Swift Testing의 모의 객체 패턴에 대해서는 `swift-protocol-di-testing` 스킬을 참조하십시오.
