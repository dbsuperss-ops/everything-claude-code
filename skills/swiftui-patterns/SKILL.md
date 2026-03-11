---
name: swiftui-patterns
description: SwiftUI 아키텍처 패턴, @Observable을 사용한 상태 관리, 뷰 구성, 내비게이션, 성능 최적화 및 최신 iOS/macOS UI 최선 관행 가이드입니다.
---

# SwiftUI 패턴 (SwiftUI Patterns)

Apple 플랫폼에서 선언적이고 성능이 우수한 사용자 인터페이스를 구축하기 위한 현대적인 SwiftUI 패턴입니다. Observation 프레임워크, 뷰 구성, 타입 안전 내비게이션 및 성능 최적화를 다룹니다.

## 활성화 시점

- SwiftUI 뷰를 설계하고 상태(`@State`, `@Observable`, `@Binding`)를 관리할 때
- `NavigationStack`을 사용하여 내비게이션 흐름을 설계할 때
- 뷰 모델과 데이터 흐름을 구조화할 때
- 리스트 및 복잡한 레이아웃의 렌더링 성능을 최적화할 때
- SwiftUI의 Environment 값과 의존성 주입(Dependency Injection)을 활용할 때

## 상태 관리 (State Management)

- **`@Observable`**: 기존의 `ObservableObject` 대신 사용하십시오. 프로퍼티 레벨에서 변경 사항을 추적하므로, 해당 프로퍼티를 읽는 뷰만 다시 렌더링되어 성능이 향상됩니다.
- **프로퍼티 래퍼 선택**: 뷰 로컬 상태는 `@State`, 부모 상태를 참조할 때는 `@Binding`, 소유한 모델은 `@State`와 함께 `@Observable` 클래스를 사용하십시오.
- **Environment 주입**: `@EnvironmentObject` 대신 `@Environment(Type.self)` 형식을 사용하여 의존성을 주입하고 사용하십시오.

## 뷰 구성 및 성능

- **서브뷰 추출**: 뷰를 작고 집중된 구조체로 쪼개십시오. 상태 변경 시 영향을 받는 서브뷰만 다시 렌더링되도록 격리하는 것이 중요합니다.
- **레이지 컨테이너 (Lazy Containers)**: 대량의 목록을 표시할 때는 `LazyVStack`이나 `LazyHStack`을 사용하여 화면에 보이는 뷰만 생성하십시오.
- **안정적인 식별자 (Stable IDs)**: `ForEach` 사용 시 배열 인덱스 대신 고유하고 변하지 않는 ID(`\.id` 또는 `Identifiable` 준수)를 사용하십시오.
- **Body 내 무거운 작업 지양**: `body` 내부에서 I/O, 네트워크 요청, 무거운 계산을 절대 수행하지 마십시오. 비동기 작업은 `.task {}`를 사용하십시오.

## 내비게이션 및 미리보기

- **타입 안전 NavigationStack**: `NavigationPath`와 `NavigationDestination`을 조합하여 프로그래밍 방식으로 안전하게 화면 전환을 처리하십시오.
- **미리보기 (#Preview)**: 인라인 모크 데이터를 활용하여 다양한 상태(데이터 없음, 로딩 중, 데이터 있음 등)를 빠르게 확인하십시오.

## 피해야 할 안티 패턴

- 새로운 코드에서 `ObservableObject`, `@Published`, `@StateObject` 사용하기 (대신 `@Observable`로 마이그레이션하십시오).
- 비동기 작업을 `body`나 `init`에서 직접 실행하기 (대신 `.task {}`를 사용하십시오).
- 조건부 뷰 처리를 위해 `AnyView` 사용하기 (대신 `@ViewBuilder`나 `Group`을 선호하십시오).

**기억하십시오**: SwiftUI는 데이터 흐름과 상태 관리가 핵심입니다. 최소한의 상태로 최대의 반응형 UI를 구현하도록 설계하십시오.
    
