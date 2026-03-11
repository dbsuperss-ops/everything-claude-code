---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Swift 테스트 (Swift Testing)

> 이 파일은 [common/testing.md](../common/testing.md)을 Swift 전용 내용으로 확장합니다.

## 프레임워크

새로운 테스트에는 **Swift Testing** (`import Testing`)을 사용하십시오. `@Test` 및 `#expect`를 활용하십시오:

```swift
@Test("유저 생성 시 이메일 유효성 검증")
func userCreationValidatesEmail() throws {
    #expect(throws: ValidationError.invalidEmail) {
        try User(email: "not-an-email")
    }
}
```

## 테스트 격리 (Test Isolation)

각 테스트는 새로운 인스턴스를 받습니다 — `init`에서 설정하고 `deinit`에서 정리하십시오. 테스트 간에 공유된 가변 상태를 두지 마십시오.

## 파라미터화된 테스트 (Parameterized Tests)

```swift
@Test("포맷 유효성 검증", arguments: ["json", "xml", "csv"])
func validatesFormat(format: String) throws {
    let parser = try Parser(format: format)
    #expect(parser.isValid)
}
```

## 커버리지

```bash
swift test --enable-code-coverage
```

## 참조

Swift Testing을 이용한 프로토콜 기반 의존성 주입 및 목(Mock) 패턴에 대해서는 `swift-protocol-di-testing` 스킬을 참조하십시오.
