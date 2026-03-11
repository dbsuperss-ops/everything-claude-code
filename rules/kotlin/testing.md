---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Kotlin 테스트 (Testing)

> 이 문서는 [common/testing.md](../common/testing.md)의 규칙을 기반으로 Kotlin 및 Android/KMP 특화 내용을 확장합니다.

## 테스트 프레임워크 (Test Framework)

- **kotlin.test**: 멀티플랫폼(KMP) 공용 테스트용 — `@Test`, `assertEquals`, `assertTrue` 등
- **JUnit 4/5**: Android 전용 테스트용
- **Turbine**: Flow 및 StateFlow 테스트용
- **kotlinx-coroutines-test**: 코루틴 테스트용 (`runTest`, `TestDispatcher` 등)

## Turbine을 이용한 ViewModel 테스트

```kotlin
@Test
fun `로딩 상태가 먼저 발행된 후 데이터가 발행되어야 함`() = runTest {
    val repo = FakeItemRepository()
    repo.addItem(testItem)
    val viewModel = ItemListViewModel(GetItemsUseCase(repo))

    viewModel.state.test {
        assertEquals(ItemListState(), awaitItem())     // 초기 상태
        viewModel.onEvent(ItemListEvent.Load)
        assertTrue(awaitItem().isLoading)               // 로딩 중
        assertEquals(listOf(testItem), awaitItem().items) // 로드 완료
    }
}
```

## 모크(Mock)보다 페이크(Fake) 선호

모킹 프레임워크를 사용하는 대신 직접 작성한 페이크(Fake) 객체 사용을 권장합니다:

```kotlin
class FakeItemRepository : ItemRepository {
    private val items = mutableListOf<Item>()
    var fetchError: Throwable? = null

    override suspend fun getAll(): Result<List<Item>> {
        fetchError?.let { return Result.failure(it) }
        return Result.success(items.toList())
    }

    override fun observeAll(): Flow<List<Item>> = flowOf(items.toList())

    fun addItem(item: Item) { items.add(item) }
}
```

## 코루틴 테스트

```kotlin
@Test
fun `병렬 작업이 성공적으로 완료되어야 함`() = runTest {
    val repo = FakeRepository()
    val result = loadDashboard(repo)
    advanceUntilIdle()
    assertNotNull(result.items)
    assertNotNull(result.stats)
}
```

항상 `runTest`를 사용하십시오. 가상 시간을 자동으로 앞당기고 `TestScope`를 제공합니다.

## Ktor MockEngine

```kotlin
val mockEngine = MockEngine { request ->
    when (request.url.encodedPath) {
        "/api/items" -> respond(
            content = Json.encodeToString(testItems),
            headers = headersOf(HttpHeaders.ContentType, ContentType.Application.Json.toString())
        )
        else -> respondError(HttpStatusCode.NotFound)
    }
}

val client = HttpClient(mockEngine) {
    install(ContentNegotiation) { json() }
}
```

## Room / SQLDelight 테스트

- **Room**: 인메모리 테스트를 위해 `Room.inMemoryDatabaseBuilder()`를 사용하십시오.
- **SQLDelight**: JVM 테스트를 위해 `JdbcSqliteDriver(JdbcSqliteDriver.IN_MEMORY)`를 사용하십시오.

```kotlin
@Test
fun `아이템 삽입 및 조회 테스트`() = runTest {
    val driver = JdbcSqliteDriver(JdbcSqliteDriver.IN_MEMORY)
    Database.Schema.create(driver)
    val db = Database(driver)

    db.itemQueries.insert("1", "Sample Item", "description")
    val items = db.itemQueries.getAll().executeAsList()
    assertEquals(1, items.size)
}
```

## 테스트 명명 규칙

백틱(backtick)을 사용하여 서술적인 이름을 작성하십시오:

```kotlin
@Test
fun `빈 쿼리로 검색 시 모든 아이템이 반환되어야 함`() = runTest { }

@Test
fun `아이템 삭제 시 삭제된 아이템이 제외된 업데이트된 목록이 발행되어야 함`() = runTest { }
```

## 테스트 디렉토리 구성

```
src/
├── commonTest/kotlin/     # 공유 테스트 (ViewModel, UseCase, Repository)
├── androidUnitTest/kotlin/ # Android 단위 테스트 (JUnit)
├── androidInstrumentedTest/kotlin/  # 기기 테스트 (Room, UI)
└── iosTest/kotlin/        # iOS 전용 테스트
```

최소 테스트 커버리지: 모든 기능에 대해 ViewModel + UseCase 테스트가 반드시 포함되어야 합니다.
