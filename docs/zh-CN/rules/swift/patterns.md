---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

# Swift 패턴 (Patterns)

> 이 문서는 [common/patterns.md](../common/patterns.md)의 내용을 바탕으로 Swift에 특화된 내용을 확장합니다.

## 프로토콜 지향 설계 (Protocol-Oriented Design)

작고 집중된 프로토콜을 정의하십시오. 프로토콜 익스텐션(Extension)을 사용하여 공통된 기본 구현을 제공하십시오:

```swift
protocol Repository: Sendable {
    associatedtype Item: Identifiable & Sendable
    func find(by id: Item.ID) async throws -> Item?
    func save(_ item: Item) async throws
}
```

## 값 타입 (Value Types)

* 데이터 전송 객체(DTO)와 모델에는 구조체(struct)를 사용하십시오.
* 서로 다른 상태를 모델링할 때는 연관 값(Associated values)을 가진 열거형(enum)을 사용하십시오:

```swift
enum LoadState<T: Sendable>: Sendable {
    case idle
    case loading
    case loaded(T)
    case failed(Error)
}
```

## 액터(Actor) 패턴

잠금(Lock)이나 디스패치 큐(DispatchQueue) 대신 공유 가변 상태를 처리하기 위해 `Actor`를 사용하십시오:

```swift
actor Cache<Key: Hashable & Sendable, Value: Sendable> {
    private var storage: [Key: Value] = [:]

    func get(_ key: Key) -> Value? { storage[key] }
    func set(_ key: Key, value: Value) { storage[key] = value }
}
```

## 의존성 주입 (Dependency Injection)

프로토콜 주입 시 기본 파라미터를 사용하십시오. 프로덕션 환경에서는 기본값을 사용하고, 테스트 시에는 모의 객체(Mock)를 주입합니다:

```swift
struct UserService {
    private let repository: any UserRepository

    init(repository: any UserRepository = DefaultUserRepository()) {
        self.repository = repository
    }
}
```

## 참고 자료

* 액터 기반의 영속성 패턴은 `swift-actor-persistence` 스킬을 참조하십시오.
* 프로토콜 기반의 의존성 주입 및 테스트 방법은 `swift-protocol-di-testing` 스킬을 참조하십시오.
