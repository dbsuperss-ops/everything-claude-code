---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Swift 코딩 스타일 (Swift Coding Style)

> 이 파일은 [common/coding-style.md](../common/coding-style.md)을 Swift 전용 내용으로 확장합니다.

## 포매팅

- 자동 포매팅을 위해 **SwiftFormat**, 스타일 강제를 위해 **SwiftLint**를 사용하십시오.
- Xcode 16+에서는 대안으로 `swift-format`이 내장되어 있습니다.

## 불변성 (Immutability)

- `var`보다 `let`을 선호하십시오 — 모든 것을 `let`으로 먼저 정의하고, 컴파일러가 요구할 때만 `var`로 변경하십시오.
- 기본적으로 값 의미론(Value semantics)을 가진 `struct`를 사용하십시오. 식별자나 참조 의미론이 필요한 경우에만 `class`를 사용하십시오.

## 명명 규칙 (Naming)

[Apple API 디자인 가이드라인](https://www.swift.org/documentation/api-design-guidelines/)을 따르십시오:

- 사용 시점의 명확성 — 불필요한 단어는 생략하십시오.
- 메서드와 프로퍼티의 이름은 타입이 아닌 역할에 따라 지으십시오.
- 전역 상수 대신 `static let`을 사용하여 상수를 정의하십시오.

## 에러 처리

타입이 지정된 throws (Swift 6+) 및 패턴 매칭을 사용하십시오:

```swift
func load(id: String) throws(LoadError) -> Item {
    guard let data = try? read(from: path) else {
        throw .fileNotFound(id)
    }
    return try decode(data)
}
```

## 동시성 (Concurrency)

Swift 6 엄격한 동시성 체크를 활성화하십시오. 다음을 권장합니다:

- 격리 경계를 넘나드는 데이터에 대해 `Sendable` 값 타입을 사용하십시오.
- 공유 가변 상태를 위해 액터(Actors)를 사용하십시오.
- 비구조적 `Task {}`보다 구조적 동시성(`async let`, `TaskGroup`)을 선호하십시오.
