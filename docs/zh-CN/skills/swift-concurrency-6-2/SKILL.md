---
name: swift-concurrency-6-2
description: Swift 6.2의 '접근 가능한 동시성(Approachable Concurrency)' 가이드입니다. 기본 싱글 스레드 동작, 명시적 백그라운드 처리를 위한 @concurrent, 메인 액터 격리 준수 패턴을 다룹니다.
origin: ECC
---

# Swift 6.2 접근 가능한 동시성 (Approachable Concurrency)

Swift 6.2 동시성 모델을 채택하기 위한 패턴입니다. 이 모델에서는 코드가 기본적으로 싱글 스레드에서 실행되며, 동시성은 명시적으로 도입됩니다. 성능 저하 없이 흔한 데이터 경쟁(Data race) 오류를 제거합니다.

## 적용 시점

* Swift 5.x 또는 6.0/6.1 프로젝트를 Swift 6.2로 마이그레이션할 때
* 데이터 경쟁 안전 관련 컴파일러 에러를 해결해야 할 때
* `MainActor` 중심의 앱 아키텍처를 설계할 때
* CPU 집약적인 작업을 백그라운드 스레드로 명시적으로 분리할 때
* Xcode 26의 'Approachable Concurrency' 빌드 설정을 활성화할 때

## 핵심 변화: 명시적 백그라운드 분리

Swift 6.1 이하 버전에서는 비동기 함수가 암시적으로 백그라운드 스레드로 넘어갈 수 있어, 안전해 보이는 코드에서도 데이터 경쟁 오류가 발생하곤 했습니다. Swift 6.2에서는 비동기 함수가 호출자(Caller)가 속한 액터(Actor)에 그대로 머무는 것이 기본 동작입니다.

```swift
// Swift 6.2: ✅ 안전 - 비동기 호출 후에도 MainActor에 유지됨
@MainActor
final class StickerModel {
    let photoProcessor = PhotoProcessor()

    func extractSticker(_ item: PhotosPickerItem) async throws -> Sticker? {
        // 호출 후에도 여전히 MainActor 상태이므로 데이터 경쟁이 발생하지 않음
        return await photoProcessor.extractSticker(data: data, with: item.id)
    }
}
```

## 핵심 패턴: 격리된 프로토콜 준수 (Isolated Conformance)

이제 `MainActor` 타입이 비격리(Non-isolated) 프로토콜을 안전하게 준수할 수 있습니다.

```swift
protocol Exportable {
    func export()
}

// Swift 6.2: @MainActor 내에서 프로토콜 구현 가능
extension StickerModel: @MainActor Exportable {
    func export() {
        photoProcessor.exportAsPNG()
    }
}
```

## 핵심 패턴: @concurrent를 통한 백그라운드 작업

진정한 병렬 처리가 필요한 경우, `@concurrent` 키워드를 사용하여 명시적으로 백그라운드로 명시하십시오.

```swift
nonisolated final class PhotoProcessor {
    // CPU 집약적인 작업을 백그라운드 스레드 풀로 명시적 분리
    @concurrent
    static func extractSubject(from data: Data) async -> Sticker {
        // 복잡한 연산 로직...
    }
}

// 호출자는 반드시 await를 사용해야 함
let sticker = await PhotoProcessor.extractSubject(from: data)
```

## 주요 설계 결정 사항

| 결정 사항 | 이유 |
|----------|-----------|
| **기본 싱글 스레드** | 가장 자연스러운 코드가 데이터 경쟁이 없는 코드임; 동시성은 선택적 도입 |
| **호출자 액터 유지** | 비동기 함수가 멋대로 백그라운드로 넘어가 발생하는 예기치 못한 에러 방지 |
| **@concurrent 명시** | 백그라운드 실행은 우연이 아닌, 성능을 위한 의도적인 선택이어야 함 |
| **MainActor 추론** | UI 앱 등에서 번거로운 `@MainActor` 어노테이션 반복을 줄임 |

## 마이그레이션 단계

1. **Xcode 설정 활성화**: Swift Compiler > Concurrency 섹션에서 설정 변경
2. **MainActor 기본값 활용**: 앱 타겟에 대해 추론 모드 활성화
3. **@concurrent 추가**: 성능 측정이 필요한 병목 구간(Hot path)에만 백그라운드 처리 적용
4. **철저한 테스트**: 데이터 경쟁 이슈가 컴파일 타임 에러로 잡히는지 확인

**핵심**: Swift 6.2는 '안전함'과 '쉬움'을 동시에 추구합니다. 우선 싱글 스레드(MainActor)에서 코드를 작성하고, 정말 필요한 경우에만 `@concurrent`로 성능을 최적화하십시오.
