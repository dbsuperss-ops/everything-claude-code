---
description: "공통 규칙을 확장하는 Swift 코딩 스타일"
globs: ["**/*.swift", "**/Package.swift"]
alwaysApply: false
---
# Swift 코딩 스타일 (Coding Style)

> 이 문서는 공통 코딩 스타일 규칙을 기반으로 Swift에 특화된 내용을 확장합니다.

## 포매팅 (Formatting)

- 자동 포매팅을 위해 **SwiftFormat**을, 스타일 강제 적용을 위해 **SwiftLint**를 사용하십시오.
- Xcode 16 이상에서는 대안으로 내장된 `swift-format`을 사용할 수 있습니다.

## 불변성 (Immutability)

- `var`보다 `let`을 선호하십시오. 모든 것을 `let`으로 정의하고, 컴파일러가 요구하는 경우에만 `var`로 변경하십시오.
- 기본적으로 값 의미론(Value semantics)을 가진 `struct`를 사용하십시오. 식별성(Identity)이나 참조 의미론이 필요한 경우에만 `class`를 사용하십시오.

## 명명 규칙 (Naming)

[Apple API 디자인 가이드라인](https://www.swift.org/documentation/api-design-guidelines/)을 따릅니다:

- 사용 시점에서의 명확성 — 불필요한 단어는 생략하십시오.
- 메서드와 속성 이름은 타입이 아닌 역할(Role)에 맞춰 명명하십시오.
- 전역 상수보다는 `static let`을 사용하여 상수를 정의하십시오.

## 에러 처리

타입이 지정된 throws(Swift 6+)와 패턴 매칭을 사용하십시오:

```swift
func load(id: String) throws(LoadError) -> Item {
    guard let data = try? read(from: path) else {
        throw .fileNotFound(id)
    }
    return try decode(data)
}
```

## 동시성 (Concurrency)

Swift 6의 엄격한 동시성 체크(Strict concurrency checking)를 활성화하십시오. 다음을 권장합니다:

- 격리 경계를 넘나드는 데이터에 대해 `Sendable` 값 타입을 사용하십시오.
- 공유되는 변경 가능한 상태(Mutable state)에 대해 액터(Actor)를 사용하십시오.
- 구조화되지 않은 `Task {}`보다 구조화된 동시성(`async let`, `TaskGroup`)을 선호하십시오.
