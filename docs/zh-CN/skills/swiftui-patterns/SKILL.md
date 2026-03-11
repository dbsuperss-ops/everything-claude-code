---
name: swiftui-patterns
description: SwiftUI 아키텍처 패턴, @Observable을 활용한 상태 관리, 뷰 조합, 내비게이션, 성능 최적화 및 현대적인 iOS/macOS UI 베스트 프랙티스입니다.
origin: ECC
---

# SwiftUI 패턴

애플 플랫폼을 위한 현대적인 SwiftUI 패턴 가이드입니다. 선언적이고 성능이 뛰어난 사용자 인터페이스를 구축하기 위한 Observation 프레임워크, 뷰 조합, 타입 안전한 내비게이션 및 성능 최적화 기법을 다룹니다.

## 적용 시점

* SwiftUI 뷰를 구축하고 상태(`@State`, `@Observable`, `@Binding`)를 관리할 때
* `NavigationStack`을 사용하여 내비게이션 흐름을 설계할 때
* 뷰 모델(ViewModel)과 데이터 흐름을 설계할 때
* 리스트 및 복잡한 레이아웃의 렌더링 성능을 최적화할 때
* 환경값(Environment values) 및 의존성 주입을 활용할 때

## 상태 관리 (Observation)

현대적인 SwiftUI(iOS 17+)에서는 `ObservableObject` 대신 `@Observable` 매크로를 사용하십시오. 이는 속성 수준에서 변경을 추적하므로, 실제 변경된 속성을 사용하는 뷰만 다시 렌더링되어 성능이 향상됩니다.

| 속성 래퍼 | 사용 시나리오 |
|---------|----------|
| `@State` | 뷰 로컬 값 타입 (토글, 텍스트 필드 값, 가벼운 상태) |
| `@Binding` | 부모 뷰의 `@State`에 대한 양방향 참조 |
| `@Observable` 클래스 + `@State` | 뷰가 소유하고 관리하는 독립적인 모델(ViewModel 등) |
| `@Observable` 클래스 (래퍼 없음) | 부모로부터 전달받은 읽기 전용 참조 |
| `@Bindable` | `@Observable` 클래스의 필드와 양방향 바인딩을 형성할 때 |
| `@Environment` | `.environment()`를 통해 주입된 공통 의존성 |

### ViewModel 예시

```swift
@Observable
final class ItemListViewModel {
    private(set) var items: [Item] = []
    private(set) var isLoading = false
    var searchText = ""

    func load() async {
        isLoading = true
        defer { isLoading = false }
        items = try? await repository.fetchAll()
    }
}
```

## 뷰 구성 및 성능 최적화

### 서브뷰 분리
뷰를 작고 전문화된 구조체로 나누십시오. 상태가 변경될 때 해당 상태를 직접 사용하는 서브뷰만 다시 렌더링되도록 격리하는 것이 성능의 핵심입니다.

### 팁: 비싼 연산 피하기
* `body` 내부에서 I/O, 네트워크 호출, 복잡한 계산을 절대 수행하지 마십시오.
* 비동기 작업은 `.task {}` 블록을 사용하십시오. 뷰가 사라지면 작업이 자동으로 취소됩니다.
* 리스트의 `ForEach`에서는 항상 고유하고 안정적인 ID를 사용하십시오. (Array index 지양)

### 내비게이션 (NavigationStack)
타입 안전한 내비게이션을 위해 `NavigationStack`과 `NavigationPath`를 함께 사용하십시오.

```swift
enum Destination: Hashable {
    case detail(Item.ID)
    case settings
}

NavigationStack(path: $router.path) {
    HomeView()
        .navigationDestination(for: Destination.self) { dest in
            switch dest {
            case .detail(let id): ItemDetailView(id: id)
            case .settings: SettingsView()
            }
        }
}
```

## 피해야 할 안티 패턴

* **구식 래퍼 사용**: 새 코드에서 `ObservableObject`, `@Published`, `@StateObject` 사용을 피하고 `@Observable`로 마이그레이션하십시오.
* **불필요한 AnyView**: 타입 정보를 지우는 `AnyView` 대신 `@ViewBuilder`나 `Group`을 사용하십시오.
* **body 내 인스턴스화**: `body` 안에서 클래스 인스턴스를 직접 생성하지 마십시오. 매번 렌더링될 때마다 새로 생성될 수 있습니다.

**핵심**: SwiftUI는 상태와 뷰의 그림자(Shadow)를 비교하여 렌더링을 결정합니다. 상태를 최소한으로 유지하고, 뷰를 작게 쪼개어 변화의 영향을 최소화하는 것이 가장 아름답고 빠른 앱을 만드는 비결입니다.
