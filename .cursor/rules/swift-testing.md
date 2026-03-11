---
description: "공통 규칙을 확장하는 Swift 테스트"
globs: ["**/*.swift", "**/Package.swift"]
alwaysApply: false
---
# Swift 테스트 (Testing)

> 이 문서는 공통 테스트 규칙을 기반으로 Swift에 특화된 내용을 확장합니다.

## 프레임워크

새로운 테스트에는 **Swift Testing**(`import Testing`)을 사용하십시오. `@Test`와 `#expect`를 활용합니다:

```swift
@Test("사용자 생성 시 이메일 유효성을 검증함")
func userCreationValidatesEmail() throws {
    #expect(throws: ValidationError.invalidEmail) {
        try User(email: "not-an-email")
    }
}
```

## 테스트 격리 (Isolation)

각 테스트는 새로운 인스턴스를 사용합니다. `init`에서 설정하고 `deinit`에서 정리하십시오. 테스트 간에 공유되는 변경 가능한 상태(Mutable state)가 없어야 합니다.

## 파라미터화된 테스트 (Parameterized Tests)

```swift
@Test("포맷 유효성 검증", arguments: ["json", "xml", "csv"])
func validatesFormat(format: String) throws {
    let parser = try Parser(format: format)
    #expect(parser.isValid)
}
```

## 커버리지 (Coverage)

```bash
swift test --enable-code-coverage
```

## 참고 자료

Swift Testing을 활용한 프로토콜 기반의 의존성 주입 및 모의 객체(Mock) 패턴에 대해서는 `swift-protocol-di-testing` 스킬을 참조하십시오.
