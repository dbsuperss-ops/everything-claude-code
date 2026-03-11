---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Kotlin 패턴 (Patterns)

> 이 문서는 [common/patterns.md](../common/patterns.md)의 규칙을 기반으로 Kotlin 및 Android/KMP(Kotlin Multiplatform)특화 내용을 확장합니다.

## 의존성 주입 (Dependency Injection)

생성자 주입(Constructor injection)을 권장합니다. KMP의 경우 **Koin**을, Android 전용의 경우 **Hilt**를 사용하십시오:

```kotlin
// Koin — 모듈 선언
val dataModule = module {
    single<ItemRepository> { ItemRepositoryImpl(get(), get()) }
    factory { GetItemsUseCase(get()) }
    viewModelOf(::ItemListViewModel)
}

// Hilt — 어노테이션 사용
@HiltViewModel
class ItemListViewModel @Inject constructor(
    private val getItems: GetItemsUseCase
) : ViewModel()
```

## ViewModel 패턴

단일 상태 객체(Single state object), 이벤트 싱크(Event sink), 단방향 데이터 흐름(UDF)을 따릅니다:

```kotlin
data class ScreenState(
    val items: List<Item> = emptyList(),
    val isLoading: Boolean = false
)

class ScreenViewModel(private val useCase: GetItemsUseCase) : ViewModel() {
    private val _state = MutableStateFlow(ScreenState())
    val state = _state.asStateFlow()

    fun onEvent(event: ScreenEvent) {
        when (event) {
            is ScreenEvent.Load -> load()
            is ScreenEvent.Delete -> delete(event.id)
        }
    }
}
```

## 저장소(Repository) 패턴

- `suspend` 함수는 `Result<T>` 또는 커스텀 에러 타입을 반환합니다.
- 반응형 스트림을 위해 `Flow`를 사용합니다.
- 로컬 데이터 소스와 리모트 데이터 소스를 조율합니다.

```kotlin
interface ItemRepository {
    suspend fun getById(id: String): Result<Item>
    suspend fun getAll(): Result<List<Item>>
    fun observeAll(): Flow<List<Item>>
}
```

## UseCase 패턴

단일 책임 원칙을 따르며, `operator fun invoke`를 활용합니다:

```kotlin
class GetItemUseCase(private val repository: ItemRepository) {
    suspend operator fun invoke(id: String): Result<Item> {
        return repository.getById(id)
    }
}

class GetItemsUseCase(private val repository: ItemRepository) {
    suspend operator fun invoke(): Result<List<Item>> {
        return repository.getAll()
    }
}
```

## expect/actual (KMP)

플랫폼별 구현을 처리하기 위해 사용합니다:

```kotlin
// commonMain
expect fun platformName(): String
expect class SecureStorage {
    fun save(key: String, value: String)
    fun get(key: String): String?
}

// androidMain
actual fun platformName(): String = "Android"
actual class SecureStorage {
    actual fun save(key: String, value: String) { /* EncryptedSharedPreferences 사용 */ }
    actual fun get(key: String): String? = null /* ... */
}

// iosMain
actual fun platformName(): String = "iOS"
actual class SecureStorage {
    actual fun save(key: String, value: String) { /* Keychain 사용 */ }
    actual fun get(key: String): String? = null /* ... */
}
```

## 코루틴(Coroutine) 패턴

- ViewModel에서는 `viewModelScope`를, 하위 작업의 구조화된 처리를 위해서는 `coroutineScope`를 사용하십시오.
- Cold Flow로부터 StateFlow를 생성할 때는 `stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), initialValue)`를 사용하십시오.
- 자식 작업의 실패가 서로에게 영향을 주지 않아야 하는 경우 `supervisorScope`를 사용하십시오.

## DSL을 활용한 빌더(Builder) 패턴

```kotlin
class HttpClientConfig {
    var baseUrl: String = ""
    var timeout: Long = 30_000
    private val interceptors = mutableListOf<Interceptor>()

    fun interceptor(block: () -> Interceptor) {
        interceptors.add(block())
    }
}

fun httpClient(block: HttpClientConfig.() -> Unit): HttpClient {
    val config = HttpClientConfig().apply(block)
    return HttpClient(config)
}

// 사용 예시
val client = httpClient {
    baseUrl = "https://api.example.com"
    timeout = 15_000
    interceptor { AuthInterceptor(tokenProvider) }
}
```

## 참고 자료

- 상세한 코루틴 패턴에 대해서는 `kotlin-coroutines-flows` 스킬을 참조하십시오.
- 모듈 및 레이어 패턴에 대해서는 `android-clean-architecture` 스킬을 참조하십시오.
