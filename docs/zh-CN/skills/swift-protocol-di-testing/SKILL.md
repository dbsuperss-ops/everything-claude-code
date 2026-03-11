---
name: swift-protocol-di-testing
description: 테스트 가능한 Swift 코드를 위한 프로토콜 기반 의존성 주입(DI) 패턴입니다. 작고 구체적인 프로토콜과 Swift Testing을 사용하여 파일 시스템, 네트워크, 외부 API를 모킹(Mocking)하는 방법을 다룹니다.
origin: ECC
---

# 프로토콜 기반 Swift 의존성 주입 테스트

파일 시스템, 네트워크, iCloud 등 외부 의존성을 작고 특정한 역할의 **프로토콜**로 추상화하여 Swift 코드를 테스트 가능하게 만드는 패턴입니다. 실제 I/O 없이도 결정론적인(Deterministic) 테스트를 수행할 수 있게 합니다.

## 적용 시점

* 파일 시스템, 네트워크, 외부 API에 접근하는 Swift 코드를 작성할 때
* 실제 환경에서 발생시키기 어려운 에러 상황(디스크 꽉 참, 네트워크 타임아웃 등)을 테스트해야 할 때
* 앱, 유닛 테스트, SwiftUI 프리뷰 등 서로 다른 환경에서 동작하는 모듈을 구축할 때
* Swift 동시성(Actor, Sendable)을 고려한 테스트 가능 아키텍처를 설계할 때

## 핵심 패턴

### 1. 작고 구체적인 프로토콜 정의
각 프로토콜은 하나의 외부 관심사만 처리해야 합니다.

```swift
// 파일 액세스 인터페이스
public protocol FileAccessorProviding: Sendable {
    func read(from url: URL) throws -> Data
    func write(_ data: Data, to url: URL) throws
    func fileExists(at url: URL) -> Bool
}
```

### 2. 프로덕션용 기본 구현체와 테스트용 모킹 구현체 생성
실제 앱에서는 `DefaultFileAccessor`를 사용하고, 테스트에서는 에러 상황을 시뮬레이션할 수 있는 `MockFileAccessor`를 사용합니다.

```swift
// 모킹 구현체 예시: 내부 딕셔너리에 데이터를 저장하여 실제 파일 쓰기 없이 테스트 가능
public final class MockFileAccessor: FileAccessorProviding, @unchecked Sendable {
    public var files: [URL: Data] = [:]
    public var readError: Error?

    public func read(from url: URL) throws -> Data {
        if let error = readError { throw error }
        guard let data = files[url] else { throw CocoaError(.fileReadNoSuchFile) }
        return data
    }
    // ... 나머지 구현
}
```

### 3. 기본 인자를 활용한 의존성 주입
생성자에서 기본값을 설정하여 평소에는 프로덕션 코드가 작동하게 하고, 테스트 시에만 모킹 객체를 주입합니다.

```swift
public actor SyncManager {
    private let fileAccessor: FileAccessorProviding

    public init(fileAccessor: FileAccessorProviding = DefaultFileAccessor()) {
        self.fileAccessor = fileAccessor
    }

    public func sync() async throws {
        let data = try fileAccessor.read(from: syncURL)
        // 싱크 로직 수행...
    }
}
```

### 4. Swift Testing을 이용한 테스트 작성

```swift
import Testing

@Test("파일 읽기 에러 발생 시 싱크 매니저의 동작 확인")
func testReadError() async {
    let mock = MockFileAccessor()
    mock.readError = CocoaError(.fileReadCorruptFile)

    let manager = SyncManager(fileAccessor: mock)

    await #expect(throws: SyncError.self) {
        try await manager.sync()
    }
}
```

## 베스트 프랙티스

* **단일 책임 원칙**: 하나의 프로토콜에 너무 많은 메서드를 넣지 마십시오. "만능 프로토콜(God Protocol)"을 피하십시오.
* **Sendable 준수**: Actor 경계를 넘나드는 프로토콜은 반드시 `Sendable`을 준수해야 합니다.
* **경계 모킹**: 내부 타입이 아닌, 외부와 맞닿아 있는 '경계(Boundary)'(파일 시스템, 네트워크 등)만 모킹하십시오.
* **불필요한 설계 지양**: 외부 의존성이 없는 단순한 타입에는 프로토콜을 도입할 필요가 없습니다.

**핵심**: 프로토콜을 통한 추상화는 코드의 유연성과 테스트 용이성을 동시에 확보하는 가장 강력한 수단입니다. 외부 시스템의 불안정함에 의존하지 않는 견고한 테스트 스위트를 구축하십시오.
