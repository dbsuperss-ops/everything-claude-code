---
name: liquid-glass-design
description: iOS 26 리퀴드 글래스(Liquid Glass) 디자인 시스템 가이드입니다. SwiftUI, UIKit, WidgetKit을 위한 동적 유리 재질, 블러, 반사 및 인터랙티브 모핑 효과 구현 방법을 다룹니다.
origin: ECC
---

# Liquid Glass 디자인 시스템 (iOS 26)

애플의 Liquid Glass 구현을 위한 패턴 가이드입니다. Liquid Glass는 하단 콘텐츠를 흐리게 만들고 주변의 색상과 빛을 반사하며, 터치 및 포인터 상호작용에 반응하는 동적 재질입니다. 본 가이드는 SwiftUI, UIKit, WidgetKit 통합 방법을 설명합니다.

## 적용 시점

* iOS 26+의 새로운 디자인 언어를 적용하여 앱을 구축하거나 업데이트할 때
* 유리 재질의 버튼, 카드, 툴바 또는 컨테이너를 구현할 때
* 유리 요소 간의 부드러운 모핑(Morphing) 전환 효과를 만들 때
* 위젯에 Liquid Glass 효과를 적용할 때
* 기존의 블러(Blur)나 매터리얼(Material) 효과를 새로운 Liquid Glass API로 마이그레이션할 때

## 핵심 패턴 — SwiftUI

### 기본 유리 효과 (Basic Glass Effect)

모든 뷰에 Liquid Glass를 적용하는 가장 간단한 방법:

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect()  // 기본값: 일반 변형(regular), 캡슐 형태
```

### 모양 및 색조 커스텀 (Custom Shape & Tint)

```swift
Text("Hello, World!")
    .font(.title)
    .padding()
    .glassEffect(.regular.tint(.orange).interactive(), in: .rect(cornerRadius: 16.0))
```

**주요 옵션:**
* `.regular` — 표준 유리 효과
* `.tint(Color)` — 강조를 위해 색조 추가
* `.interactive()` — 터치 및 포인터 상호작용에 반응
* 모양(Shape): `.capsule`(기본), `.rect(cornerRadius:)`, `.circle`

### 유리 버튼 스타일

```swift
Button("클릭하세요") { /* 액션 */ }
    .buttonStyle(.glass)

Button("중요") { /* 액션 */ }
    .buttonStyle(.glassProminent)
```

### GlassEffectContainer 활용

성능 최적화와 모핑 효과를 위해, 여러 개의 유리 뷰는 항상 하나의 컨테이너로 감싸십시오:

```swift
GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .frame(width: 80.0, height: 80.0)
            .glassEffect()

        Image(systemName: "eraser.fill")
            .frame(width: 80.0, height: 80.0)
            .glassEffect()
    }
}
```
* `spacing` 파라미터는 병합 거리(Merge distance)를 조절합니다. 요소들이 가까워지면 유리 모양이 하나로 합쳐집니다.

### 모핑 전환 (Morphing Transitions)

유리 요소가 나타나거나 사라질 때 부드러운 모핑 효과를 생성합니다:

```swift
@State private var isExpanded = false
@Namespace private var namespace

GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "pencil")
            .glassEffect()
            .glassEffectID("pencil", in: namespace)

        if isExpanded {
            Image(systemName: "eraser")
                .glassEffect()
                .glassEffectID("eraser", in: namespace)
        }
    }
}

Button("전환") {
    withAnimation { isExpanded.toggle() }
}
.buttonStyle(.glass)
```

## 핵심 패턴 — UIKit

### 기본 UIGlassEffect

```swift
let glassEffect = UIGlassEffect()
glassEffect.tintColor = UIColor.systemBlue.withAlphaComponent(0.3)
glassEffect.isInteractive = true

let visualEffectView = UIVisualEffectView(effect: glassEffect)
visualEffectView.layer.cornerRadius = 20
visualEffectView.clipsToBounds = true

view.addSubview(visualEffectView)
// 제약 조건(Constraints) 설정...
```

### 다중 요소용 UIGlassContainerEffect

```swift
let containerEffect = UIGlassContainerEffect()
containerEffect.spacing = 40.0

let containerView = UIVisualEffectView(effect: containerEffect)

let firstGlass = UIVisualEffectView(effect: UIGlassEffect())
let secondGlass = UIVisualEffectView(effect: UIGlassEffect())

containerView.contentView.addSubview(firstGlass)
containerView.contentView.addSubview(secondGlass)
```

## 핵심 패턴 — WidgetKit

### 렌더링 모드 감지

```swift
struct MyWidgetView: View {
    @Environment(\.widgetRenderingMode) var renderingMode

    var body: some View {
        if renderingMode == .accented {
            // 강조 모드: 화이트 틴트가 적용된 테마 유리 배경
        } else {
            // 풀 컬러 모드: 표준 외관
        }
    }
}
```

### 위젯 컨테이너 배경

```swift
VStack { /* 콘텐츠 */ }
    .containerBackground(for: .widget) {
        Color.blue.opacity(0.2)
    }
```

## 주요 설계 결정 사항

| 결정 사항 | 사유 |
|----------|-----------|
| **GlassEffectContainer 사용** | 성능 최적화 및 유리 요소 간의 모핑 구현을 위해 필수 |
| **spacing 파라미터** | 병합 거리를 제어하여 요소가 얼마나 가까워야 합쳐질지 미세 조정 |
| **@Namespace + glassEffectID** | 뷰 계층 구조가 변할 때 부드러운 모핑 전환 구현 |
| **interactive() 수식어** | 터치 반응 여부 선택 — 모든 유리가 반응할 필요는 없음 |
| **accented 렌더링 모드** | 사용자가 홈 화면 테마를 변경했을 때 시스템 테마와 조화되도록 지원 |

## 베스트 프랙티스

* 여러 형제 뷰에 유리 효과를 적용할 때는 **항상 `GlassEffectContainer`를 사용**하십시오. 모핑을 지원하고 렌더링 성능을 높여줍니다.
* 외관 관련 수식어(frame, font, padding)를 먼저 적용한 **후에 `.glassEffect()`를 적용**하십시오.
* **사용자 상호작용이 발생하는 요소**(버튼, 토글 등)에만 `.interactive()`를 사용하십시오.
* 뷰 계층이 바뀔 때 부드러운 모핑을 위해 **`withAnimation` 내에서 상태를 변경**하십시오.
* **다양한 외관 모드**(라이트, 다크, 강조 모드)에서 가독성과 대비(Contrast)를 테스트하십시오.

## 피해야 할 안티 패턴

* `GlassEffectContainer` 없이 독립적인 `.glassEffect()` 뷰를 남발하는 행위
* 유리 효과를 너무 많이 중첩하는 행위 (성능 저하 및 시각적 혼란 초래)
* 모든 뷰에 무차별적으로 유리 효과를 적용하는 행위 (인터랙션 요소, 툴바, 카드 등에 예약 권장)
* UIKit에서 `clipsToBounds = true` 설정을 누락하여 모서리 반경이 적용되지 않는 문제
* 유리 효과 뒤에 불투명한 배경을 두어 반투명 효과를 무용지물로 만드는 행위
* 위젯의 `accented` 렌더링 모드를 무시하여 시스템 테마 조화를 깨뜨리는 행위

## 사용 사례

* iOS 26 디자인 언어를 따르는 내비게이션 바, 툴바, 탭 바
* 플로팅 액션 버튼(FAB) 및 카드형 컨테이너
* 시각적 깊이와 터치 피드백이 필요한 상호작용 컨트롤
* 시스템 Liquid Glass 외관과 통합되어야 하는 위젯
* UI 상태 변화에 따른 유기적인 모핑 전환 효과
