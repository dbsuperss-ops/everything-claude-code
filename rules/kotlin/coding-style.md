---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Kotlin 코딩 스타일 (Coding Style)

> 이 문서는 [common/coding-style.md](../common/coding-style.md)의 규칙을 기반으로 Kotlin에 특화된 내용을 확장합니다.

## 포매팅 (Formatting)

- 스타일 강제 적용을 위해 **ktlint** 또는 **Detekt**를 사용하십시오.
- 공식 Kotlin 코드 스타일을 따릅니다 (`gradle.properties` 내 `kotlin.code.style=official` 설정).

## 불변성 (Immutability)

- `var`보다 `val`을 선호하십시오. 기본적으로 `val`로 정의하고, 상태 변경이 반드시 필요한 경우에만 `var`로 변경하십시오.
- 값 타입에는 `data class`를 사용하십시오. 공개 API에서는 불변 컬렉션(`List`, `Map`, `Set`)을 사용하십시오.
- 상태 업데이트 시 Copy-on-write 방식을 사용하십시오: `state.copy(field = newValue)`

## 명명 규칙 (Naming)

Kotlin 컨벤션을 따릅니다:
- 함수 및 속성: `camelCase`
- 클래스, 인터페이스, 객체, 타입 별칭: `PascalCase`
- 상수 (`const val` 또는 `@JvmStatic`): `SCREAMING_SNAKE_CASE`
- 인터페이스 접두사로 `I`를 붙이지 말고 행위 중심으로 명명하십시오: `IClickable` 대신 `Clickable`

## Null 안전성 (Null Safety)

- 절대로 `!!`를 사용하지 마십시오. 대신 `?.`, `?:`, `requireNotNull()` 또는 `checkNotNull()`을 사용하십시오.
- 범위가 한정된 null 안전 작업에는 `?.let {}`을 사용하십시오.
- 결과가 없을 수 있는 함수의 경우 Nullable 타입을 반환하십시오.

```kotlin
// 잘못된 예
val name = user!!.name

// 올바른 예
val name = user?.name ?: "Unknown"
val name = requireNotNull(user) { "이름에 접근하기 전 사용자가 설정되어야 합니다" }.name
```

## 봉인된 타입 (Sealed Types)

닫힌 상태 계층(Closed state hierarchies)을 모델링하려면 sealed 클래스/인터페이스를 사용하십시오:

```kotlin
sealed interface UiState<out T> {
    data object Loading : UiState<Nothing>
    data class Success<T>(val data: T) : UiState<T>
    data class Error(val message: String) : UiState<Nothing>
}
```

sealed 타입을 사용할 때는 항상 `when` 구문을 사용하여 모든 경우를 명시적으로 처리하십시오 (else 브랜치 사용 금지).

## 확장 함수 (Extension Functions)

유틸리티 작업을 위해 확장 함수를 사용하되, 가시성을 유지하십시오:
- 수신 객체 타입의 이름을 딴 파일에 위치시키십시오 (`StringExt.kt`, `FlowExt.kt` 등).
- 범위를 제한적으로 유지하십시오. `Any`나 지나치게 제네릭한 타입에 확장을 추가하지 마십시오.

## 범위 지정 함수 (Scope Functions)

목적에 맞는 범위 지정 함수를 사용하십시오:
- `let` — null 체크 + 변환: `user?.let { greet(it) }`
- `run` — 수신 객체를 사용하여 결과 계산: `service.run { fetch(config) }`
- `apply` — 객체 구성/설정: `builder.apply { timeout = 30 }`
- `also` — 부수 효과(Side effects) 수행: `result.also { log(it) }`
- 범위 지정 함수를 깊게 중첩하지 마십시오 (최대 2단계).

## 에러 처리

- `Result<T>` 또는 커스텀 sealed 타입을 사용하십시오.
- 예외가 발생할 수 있는 코드를 래핑할 때는 `runCatching {}`을 사용하십시오.
- `CancellationException`은 절대 catch하지 마십시오. 항상 다시 던져야(Rethrow) 합니다.
- 제어 흐름(Control flow)을 위해 `try-catch`를 사용하지 마십시오.

```kotlin
// 잘못된 예 — 제어 흐름을 위해 예외 사용
val user = try { repository.getUser(id) } catch (e: NotFoundException) { null }

// 올바른 예 — Nullable 반환 사용
val user: User? = repository.findUser(id)
```
