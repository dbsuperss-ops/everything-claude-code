---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Swift 패턴 (Swift Patterns)

> 이 파일은 [common/patterns.md](../common/patterns.md)을 Swift 전용 내용으로 확장합니다.

## 프로토콜 지향 설계 (Protocol-Oriented Design)

작고 집중된 프로토콜을 정의하십시오. 공통 기본 동작을 위해 프로토콜 익스텐션(Extension)을 사용하십시오:

```swift
protocol Repository: Sendable {
    associatedtype Item: Identifiable & Sendable
    func find(by id: Item.ID) async throws -> Item?
    func save(_ item: Item) async throws
}
```

## 값 타입 (Value Types)

- 데이터 전송 객체(DTO)와 모델에 `struct`를 사용하십시오.
- 서구분된 상태를 모델링하기 위해 연관 값(Associated values)을 가진 `enum`을 사용하십시오:

```swift
enum LoadState<T: Sendable>: Sendable {
    case idle
    case loading
    case loaded(T)
    case failed(Error)
}
```

## 액터 패턴 (Actor Pattern)

잠금(Lock)이나 디스패치 큐 대신 공유 가변 상태를 위해 액터(Actors)를 사용하십시오:

```swift
actor Cache<Key: Hashable & Sendable, Value: Sendable> {
    private var storage: [Key: Value] = [:]

    func get(_ key: Key) -> Value? { storage[key] }
    func set(_ key: Key, value: Value) { storage[key] = value }
}
```

## 의존성 주입 (Dependency Injection)

기본 매개변수와 함께 프로토콜을 주입하십시오 — 프로덕션에서는 기본값을 사용하고, 테스트에서는 목(Mock)을 주입합니다.

```swift
struct UserService {
    private let repository: any UserRepository

    init(repository: any UserRepository = DefaultUserRepository()) {
        self.repository = repository
    }
}
```

## 참조

액터 기반 퍼시스턴스 패턴에 대해서는 `swift-actor-persistence` 스킬을 참조하십시오.
프로토콜 기반 DI 및 테스트에 대해서는 `swift-protocol-di-testing` 스킬을 참조하십시오.
