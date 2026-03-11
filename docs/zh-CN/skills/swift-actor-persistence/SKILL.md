---
name: swift-actor-persistence
description: Swift에서 actor를 사용하여 스레드 안전한 데이터 퍼시스턴스를 구현합니다. 메모리 내 캐시와 파일 기반 저장소를 결합하여 설계 단계에서 데이터 경쟁(Data race)을 제거합니다.
origin: ECC
---

# 스레드 안전한 퍼시스턴스를 위한 Swift Actor

Swift **actor**를 사용하여 스레드 안전한 데이터 퍼시스턴스 레이어를 구축하는 패턴입니다. 메모리 내 캐시와 파일 기반 저장소를 결합하고, actor 모델을 활용하여 컴파일 타임에 데이터 경쟁을 제거합니다.

## 적용 시점

* Swift 5.5 이상에서 데이터 퍼시스턴스 레이어를 구축할 때
* 공유 가변 상태(Shared mutable state)에 대한 스레드 안전한 접근이 필요할 때
* 수동 동기화(Lock, DispatchQueue 등)를 제거하고 싶을 때
* 로컬 저장소를 갖춘 오프라인 우선(Offline-first) 앱을 구축할 때

## 핵심 패턴: Actor 기반 저장소 (Repository)

Actor 모델은 직렬화된 접근(Serialized access)을 보장합니다. 즉, 데이터 경쟁이 발생하지 않으며 이는 컴파일러에 의해 강제됩니다.

```swift
public actor LocalRepository<T: Codable & Identifiable> where T.ID == String {
    private var cache: [String: T] = [:]
    private let fileURL: URL

    public init(directory: URL = .documentsDirectory, filename: String = "data.json") {
        self.fileURL = directory.appendingPathComponent(filename)
        // 초기화 중에는 actor 격리가 활성화되지 않으므로 동기 로드 가능
        self.cache = Self.loadSynchronously(from: fileURL)
    }

    // 데이터 저장: 캐시 업데이트 후 파일에 기록
    public func save(_ item: T) throws {
        cache[item.id] = item
        try persistToFile()
    }

    // 데이터 조회: 메모리 캐시에서 즉시 반환
    public func find(by id: String) -> T? {
        cache[id]
    }

    private func persistToFile() throws {
        let data = try JSONEncoder().encode(Array(cache.values))
        try data.write(to: fileURL, options: .atomic)
    }
}
```

## 사용 방법

Actor 격리 덕분에 모든 호출은 자동으로 비동기 처리가 됩니다 (`await` 필요):

```swift
let repository = LocalRepository<Question>()

// 조회 - 메모리 캐시를 통한 빠른 O(1) 검색
let question = await repository.find(by: "q-001")

// 저장 - 캐시 업데이트 및 원자적(Atomic) 파일 기록
try await repository.save(newQuestion)
```

## 주요 설계 결정 사항

| 결정 사항 | 이유 |
|----------|-----------|
| **Actor 사용** | 수동 동기화 없이 컴파일러 수준에서 스레드 안전성 보장 |
| **메모리 캐시 + 파일 저장** | 읽기는 캐시로 빠르게, 쓰기는 파일로 안전하게 처리 |
| **동기 초기화 로드** | 비동기 초기화의 복잡성을 피하고 초기 구동 속도 확보 |
| **원자적 쓰기 (.atomic)** | 쓰기 도중 앱 종료 시 데이터 손상을 방지 |

## 베스트 프랙티스

* **Sendable 타입 활용**: Actor 경계를 넘나드는 모든 데이터는 `Sendable` 프로토콜을 준수해야 합니다.
* **최소 노출**: Actor의 공용 API를 최소화하고 퍼시스턴스 상세 구현(파일 경로 등)은 숨기십시오.
* **@Observable 연동**: SwiftUI 환경에서는 `@Observable` ViewModel과 결합하여 반응형 UI 업데이트를 구현하십시오.
* **Lock 대신 Actor 사용**: 현대적인 Swift 동시성을 사용하여 `DispatchQueue`나 `NSLock` 의존성을 줄이십시오.

**핵심**: Swift Actor는 안전한 데이터 관리를 위한 가장 현대적인 도구입니다. 복잡한 락 관리 대신 언어 차원의 제약 조건(Actor)을 활용하여 버그 없는 데이터 레이어를 구축하십시오.
