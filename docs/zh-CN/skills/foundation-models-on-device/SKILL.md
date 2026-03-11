---
name: foundation-models-on-device
description: 기기용 LLM을 위한 애플의 FoundationModels 프레임워크 가이드입니다. 텍스트 생성, @Generable을 사용한 제어 생성, 도구 호출(Tool calling) 및 iOS 26+의 스냅샷 스트리밍을 다룹니다.
origin: ECC
---

# FoundationModels: 온디바이스(On-device) LLM (iOS 26)

애플의 온디바이스 언어 모델을 앱에 통합하기 위한 FoundationModels 프레임워크 활용 패턴입니다. 텍스트 생성, `@Generable`을 이용한 구조화된 출력, 커스텀 도구 호출 및 스냅샷 스트리밍을 다루며, 모든 작업은 개인정보 보호와 오프라인 지원을 위해 기기 내부에서 실행됩니다.

## 적용 시점

* Apple Intelligence를 사용하여 온디바이스 AI 기능을 구축할 때
* 클라우드 의존 없이 텍스트 생성 또는 요약 기능을 구현할 때
* 자연어 입력에서 구조화된 데이터를 추출할 때
* 특정 도메인용 AI 작업을 위해 커스텀 도구 호출(Tool calling)을 구현할 때
* 실시간 UI 업데이트를 위해 구조화된 응답을 스트리밍할 때
* 데이터가 기기를 벗어나지 않는 개인정보 보호 중심 AI가 필요할 때

## 핵심 패턴 — 가용성 확인 (Availability Check)

세션을 생성하기 전에 반드시 모델의 가용성을 확인하십시오:

```swift
struct GenerativeView: View {
    private var model = SystemLanguageModel.default

    var body: some View {
        switch model.availability {
        case .available:
            ContentView()
        case .unavailable(.deviceNotEligible):
            Text("Apple Intelligence를 지원하지 않는 기기입니다.")
        case .unavailable(.appleIntelligenceNotEnabled):
            Text("설정에서 Apple Intelligence를 활성화해주세요.")
        case .unavailable(.modelNotReady):
            Text("모델 다운로드 중이거나 준비되지 않았습니다.")
        case .unavailable(let other):
            Text("모델 사용 불가: \(other)")
        }
    }
}
```

## 핵심 패턴 — 기본 세션 (Basic Session)

```swift
// 단발성(Single-turn): 매번 새로운 세션 생성
let session = LanguageModelSession()
let response = try await session.respond(to: "파리를 방문하기 좋은 달은 언제인가요?")
print(response.content)

// 다회성(Multi-turn): 대화 맥락 유지를 위해 세션 재사용
let session = LanguageModelSession(instructions: """
    당신은 요리 보조원입니다.
    재료에 기반한 레시피 제안을 제공하십시오.
    제안은 간결하고 실용적이어야 합니다.
    """)

let first = try await session.respond(to: "닭고기와 쌀이 있습니다.")
let followUp = try await session.respond(to: "채식 옵션은 어떤 게 있을까요?")
```

**지침(Instructions) 작성 팁:**
* 모델의 역할 정의 ("당신은 튜터입니다")
* 수행할 작업 명시 ("캘린더 일정 추출을 도와주세요")
* 스타일 선호도 설정 ("가능한 짧게 답하세요")
* 안전 장치 추가 ("위험한 요청에는 '도와드릴 수 없습니다'라고 답하세요")

## 핵심 패턴 — @Generable을 이용한 제어 생성

원시 문자열 대신 구조화된 Swift 타입을 생성합니다:

### 1. 생성 가능한 타입 정의
```swift
@Generable(description: "고양이에 대한 기본 프로필 정보")
struct CatProfile {
    var name: String

    @Guide(description: "고양이의 나이", .range(0...20))
    var age: Int

    @Guide(description: "고양이의 성격에 대한 한 문장 소개")
    var profile: String
}
```

### 2. 구조화된 출력 요청
```swift
let response = try await session.respond(
    to: "귀여운 유기묘 한 마리를 생성해줘",
    generating: CatProfile.self
)

// 구조화된 필드에 직접 접근
print("이름: \(response.content.name)")
print("나이: \(response.content.age)")
print("소개: \(response.content.profile)")
```

**지원되는 @Guide 제약 조건:**
* `.range(0...20)` — 수치 범위 제한
* `.count(3)` — 배열 요소 개수 제한
* `description:` — 생성 방향에 대한 의미적 가이드

## 핵심 패턴 — 도구 호출 (Tool Calling)

모델이 특정 도메인 작업을 수행하기 위해 커스텀 코드를 호출하게 합니다:

### 1. 도구 정의
```swift
struct RecipeSearchTool: Tool {
    let name = "recipe_search"
    let description = "주어진 용어와 일치하는 레시피를 검색하고 결과 목록을 반환합니다."

    @Generable
    struct Arguments {
        var searchTerm: String
        var numberOfResults: Int
    }

    func call(arguments: Arguments) async throws -> ToolOutput {
        let recipes = await searchRecipes(
            term: arguments.searchTerm,
            limit: arguments.numberOfResults
        )
        return .string(recipes.map { "- \($0.name): \($0.description)" }.joined(separator: "\n"))
    }
}
```

### 2. 도구가 포함된 세션 생성
```swift
let session = LanguageModelSession(tools: [RecipeSearchTool()])
let response = try await session.respond(to: "파스타 레시피 좀 찾아줘")
```

## 핵심 패턴 — 스냅샷 스트리밍 (Snapshot Streaming)

실시간 UI를 위해 `PartiallyGenerated` 타입을 사용하여 구조화된 응답을 스트리밍합니다:

```swift
@Generable
struct TripIdeas {
    @Guide(description: "추천 여행 아이디어 목록")
    var ideas: [String]
}

let stream = session.streamResponse(
    to: "흥미로운 여행지 아이디어를 알려줘",
    generating: TripIdeas.self
)

for try await partial in stream {
    // partial: TripIdeas.PartiallyGenerated (모든 속성이 Optional임)
    print(partial)
}
```

### SwiftUI 통합 예시
```swift
@State private var partialResult: TripIdeas.PartiallyGenerated?

var body: some View {
    List(partialResult?.ideas ?? [], id: \.self) { idea in
        Text(idea)
    }
    .task {
        let stream = session.streamResponse(to: prompt, generating: TripIdeas.self)
        for try await partial in stream {
            partialResult = partial
        }
    }
}
```

## 주요 설계 결정 사항

| 결정 사항 | 사유 |
|----------|-----------|
| **온디바이스 실행** | 개인정보 보호(데이터 유출 방지), 오프라인 지원 |
| **4,096 토큰 제한** | 온디바이스 모델의 한계; 큰 데이터는 세션을 나누어 처리 권장 |
| **스냅샷 스트리밍** | 구조화된 출력에 적합; 각 스냅샷은 완성된 부분 상태를 제공 |
| **@Generable 매크로** | 구조화된 생성에 대한 컴파일 시점 안전성 보장; `PartiallyGenerated` 타입 자동 생성 |
| **세션당 단일 요청** | `isResponding` 속성으로 동시 요청 방지; 필요 시 다수 세션 생성 |
| **response.content** | 결과 접근을 위해 항상 `.output` 대신 `.content` 속성 사용 |

## 베스트 프랙티스

* 세션 생성 전 **항상 `model.availability`를 확인**하여 사용 불가 상황에 대응하십시오.
* 모델 동작 제어를 위해 **`instructions`를 사용**하십시오 (프롬프트보다 우선순위가 높음).
* 새로운 요청을 보내기 전 **`isResponding` 상태를 확인**하십시오. 세션은 한 번에 하나의 요청만 처리합니다.
* 응답 데이터 접근 시 `.output` 대신 **`response.content`를 사용**하십시오.
* **대량의 입력은 나누어서 처리**하십시오. 4,096 토큰 제한은 지침, 프롬프트, 출력을 모두 합친 크기입니다.
* 구조화된 출력이 필요하면 **`@Generable`을 적극 활용**하십시오 (원시 문자열 파싱보다 안전함).
* 창의성 조절을 위해 **`GenerationOptions(temperature:)`를 사용**하십시오 (높을수록 창의적).
* Xcode의 **Instruments 도구**를 사용하여 요청 성능을 분석하십시오.

## 피해야 할 안티 패턴

* `model.availability` 확인 없이 세션을 생성하는 행위
* 4,096 토큰 컨텍스트 윈도우를 초과하는 입력을 보내는 행위
* 단일 세션에서 동시에 여러 요청을 시도하는 행위
* 응답 데이터 접근에 `.content` 대신 `.output`을 사용하는 행위
* 구조화된 출력이 가능한 상황에서 굳이 원시 문자열 응답을 파싱하려 하는 행위
* 단일 프롬프트에 너무 복잡한 다단계 로직을 넣는 행위 (작고 집중된 프롬프트로 분리 권장)
* 모델이 항상 사용 가능하다고 가정하는 행위 (기기 사양 및 설정에 따라 다름)
