---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

# Swift 코딩 스타일 (Coding Style)

> 이 문서는 [common/coding-style.md](../common/coding-style.md)의 내용을 바탕으로 Swift에 특화된 내용을 확장합니다.

## 포매팅 (Formatting)

* 자동 포매팅은 **SwiftFormat**, 스타일 검사는 **SwiftLint**를 사용합니다.
* Xcode 16+ 버전부터는 `swift-format`이 대체 옵션으로 기본 내장되어 있습니다.

## 불변성 (Immutability)

* `var`보다 `let` 사용을 우선시하십시오. 모든 것을 `let`으로 정의하고, 컴파일러가 요구할 때만 `var`로 변경하십시오.
* 값 세만틱(Value semantics)을 가진 `struct` 사용을 기본으로 하되, 식별자(Identity)나 참조 세만틱이 필요한 경우에만 `class`를 사용하십시오.

## 명명 규칙 (Naming)

[Apple API 디자인 가이드라인](https://www.swift.org/documentation/api-design-guidelines/)을 따릅니다:

* 사용 시점에 명확해야 합니다 — 불필요한 단어는 생략하십시오.
* 타입이 아닌 메서드와 속성의 **역할**을 중심으로 이름을 지으십시오.
* 상수의 경우 전역 상수 대신 `static let`을 사용하십시오.

## 에러 처리

Swift 6+의 타입화된 throws(Typed throws)와 패턴 매칭을 활용하십시오:

```swift
func load(id: String) throws(LoadError) -> Item {
    guard let data = try? read(from: path) else {
        throw .fileNotFound(id)
    }
    return try decode(data)
}
```

## 동시성 (Concurrency)

Swift 6의 엄격한 동시성 체크(Strict Concurrency Checking)를 활성화하십시오. 다음을 권장합니다:

* 격리 경계를 넘나드는 데이터에 `Sendable` 값 타입을 사용하십시오.
* 공유되는 가변 상태를 관리하기 위해 `Actor`를 사용하십시오.
* 비구조적 `Task {}`보다는 구조적 동시성(`async let`, `TaskGroup`)을 우선 사용하십시오.
