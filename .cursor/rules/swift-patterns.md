---
description: "공통 규칙을 확장하는 Swift 패턴"
globs: ["**/*.swift", "**/Package.swift"]
alwaysApply: false
---
# Swift 패턴 (Patterns)

> 이 문서는 공통 패턴 규칙을 기반으로 Swift에 특화된 내용을 확장합니다.

## 프로토콜 지향 설계 (Protocol-Oriented Design)

작고 집중된 프로토콜을 정의하십시오. 공통된 기본 구현을 위해 프로토콜 확장을 사용하십시오:

```swift
protocol Repository: Sendable {
    associatedtype Item: Identifiable & Sendable
    func find(by id: Item.ID) async throws -> Item?
    func save(_ item: Item) async throws
}
```

## 값 타입 (Value Types)

- 데이터 전송 객체(DTO) 및 모델에 구조체(Struct)를 사용하십시오.
- 서로 다른 상태를 모델링하려면 연관 값(Associated values)이 포함된 열거형(Enum)을 사용하십시오:

```swift
enum LoadState<T: Sendable>: Sendable {
    case idle
    case loading
    case loaded(T)
    case failed(Error)
}
```

## 액터 패턴 (Actor Pattern)

잠금(Lock)이나 디스패치 큐(Dispatch queues) 대신, 공유되는 변경 가능한 상태를 관리하기 위해 액터(Actor)를 사용하십시오:

```swift
actor Cache<Key: Hashable & Sendable, Value: Sendable> {
    private var storage: [Key: Value] = [:]

    func get(_ key: Key) -> Value? { storage[key] }
    func set(_ key: Key, value: Value) { storage[key] = value }
}
```

## 의존성 주입 (Dependency Injection)

기본 파라미터와 함께 프로토콜을 주입하십시오. 프로덕션 코드에서는 기본값을 사용하고, 테스트에서는 모의 객체(Mock)를 주입합니다:

```swift
struct UserService {
    private let repository: any UserRepository

    init(repository: any UserRepository = DefaultUserRepository()) {
        self.repository = repository
    }
}
```

## 참고 자료

액터 기반 영속성 패턴에 대해서는 `swift-actor-persistence` 스킬을 참조하십시오.
프로토콜 기반의 의존성 주입 및 테스트에 대해서는 `swift-protocol-di-testing` 스킬을 참조하십시오.
