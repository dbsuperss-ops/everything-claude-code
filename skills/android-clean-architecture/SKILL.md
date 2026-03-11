---
name: android-clean-architecture
description: Android 및 Kotlin Multiplatform(KMP) 프로젝트를 위한 클린 아키텍처 패턴 — 모듈 구조, 의존성 규칙, UseCase, Repository 및 데이터 레이어 패턴을 포함합니다.
origin: ECC
---

# Android 클린 아키텍처 (Android Clean Architecture)

Android 및 KMP 프로젝트를 위한 클린 아키텍처 패턴 가이드입니다. 모듈 경계, 의존성 역전, UseCase/Repository 패턴, 그리고 Room, SQLDelight, Ktor를 사용한 데이터 레이어 설계를 다룹니다.

## 활성화 시기

- Android 또는 KMP 프로젝트 모듈 구조를 잡을 때
- UseCase, Repository 또는 DataSource를 구현할 때
- 레이어 간(domain, data, presentation) 데이터 흐름을 설계할 때
- Koin 또는 Hilt를 사용하여 의존성 주입(DI)을 설정할 때
- 계층화된 아키텍처 내에서 Room, SQLDelight 또는 Ktor로 작업할 때

## 모듈 구조 (Module Structure)

### 권장 레이아웃

```
project/
├── app/                  # Android 진입점, DI 구성, Application 클래스
├── core/                 # 공통 유틸리티, 베이스 클래스, 에러 타입
├── domain/               # UseCase, 도메인 모델, 리포지토리 인터페이스 (순수 Kotlin)
├── data/                 # 리포지토리 구현체, DataSource, DB, 네트워크 처리
├── presentation/         # 화면(Screens), ViewModel, UI 모델, 내비게이션
├── design-system/        # 재사용 가능한 Compose 컴포넌트, 테마, 타이포그래피
└── feature/              # 기능별 모듈 (선택 사항, 규모가 큰 프로젝트용)
    ├── auth/
    ├── settings/
    └── profile/
```

### 의존성 규칙

```
app → presentation, domain, data, core
presentation → domain, design-system, core
data → domain, core
domain → core (또는 의존성 없음)
core → (의존성 없음)
```

**중요**: `domain` 레이어는 절대로 `data`, `presentation` 또는 특정 프레임워크에 의존해서는 안 됩니다. 순수하게 Kotlin 코드로만 구성되어야 합니다.

## 도메인 레이어 (Domain Layer)

### UseCase 패턴

각 UseCase는 하나의 비즈니스 작업을 나타냅니다. 호출을 깔끔하게 하기 위해 `operator fun invoke`를 사용하십시오.

```kotlin
class GetItemsByCategoryUseCase(
    private val repository: ItemRepository
) {
    suspend operator fun invoke(category: String): Result<List<Item>> {
        return repository.getItemsByCategory(category)
    }
}

// 반응형 스트림을 위한 Flow 기반 UseCase
class ObserveUserProgressUseCase(
    private val repository: UserRepository
) {
    operator fun invoke(userId: String): Flow<UserProgress> {
        return repository.observeProgress(userId)
    }
}
```

### 도메인 모델 (Domain Models)

도메인 모델은 프레임워크 어노테이션이 없는 일반적인 Kotlin 데이터 클래스여야 합니다.

```kotlin
data class Item(
    val id: String,
    val title: String,
    val description: String,
    val tags: List<String>,
    val status: Status,
    val category: String
)

enum class Status { DRAFT, ACTIVE, ARCHIVED }
```

### 리포지토리 인터페이스 (Repository Interfaces)

도메인 레이어에서 정의하고, 데이터 레이어에서 구현합니다.

```kotlin
interface ItemRepository {
    suspend fun getItemsByCategory(category: String): Result<List<Item>>
    suspend fun saveItem(item: Item): Result<Unit>
    fun observeItems(): Flow<List<Item>>
}
```

## 데이터 레이어 (Data Layer)

### 리포지토리 구현체

로컬 및 원격 데이터 소스 간의 조정을 담당합니다.

```kotlin
class ItemRepositoryImpl(
    private val localDataSource: ItemLocalDataSource,
    private val remoteDataSource: ItemRemoteDataSource
) : ItemRepository {

    override suspend fun getItemsByCategory(category: String): Result<List<Item>> {
        return runCatching {
            val remote = remoteDataSource.fetchItems(category)
            localDataSource.insertItems(remote.map { it.toEntity() })
            localDataSource.getItemsByCategory(category).map { it.toDomain() }
        }
    }

    override suspend fun saveItem(item: Item): Result<Unit> {
        return runCatching {
            localDataSource.insertItems(listOf(item.toEntity()))
        }
    }

    override fun observeItems(): Flow<List<Item>> {
        return localDataSource.observeAll().map { entities ->
            entities.map { it.toDomain() }
        }
    }
}
```

### 매퍼(Mapper) 패턴

데이터 모델 근처에 확장 함수로 매퍼를 작성하십시오.

```kotlin
// 데이터 레이어 내에서
fun ItemEntity.toDomain() = Item(
    id = id,
    title = title,
    description = description,
    tags = tags.split("|"),
    status = Status.valueOf(status),
    category = category
)

fun ItemDto.toEntity() = ItemEntity(
    id = id,
    title = title,
    description = description,
    tags = tags.joinToString("|"),
    status = status,
    category = category
)
```

### Room 데이터베이스 (Android)

```kotlin
@Entity(tableName = "items")
data class ItemEntity(
    @PrimaryKey val id: String,
    val title: String,
    val description: String,
    val tags: String,
    val status: String,
    val category: String
)

@Dao
interface ItemDao {
    @Query("SELECT * FROM items WHERE category = :category")
    suspend fun getByCategory(category: String): List<ItemEntity>

    @Upsert
    suspend fun upsert(items: List<ItemEntity>)

    @Query("SELECT * FROM items")
    fun observeAll(): Flow<List<ItemEntity>>
}
```

### SQLDelight (KMP)

```sql
-- Item.sq
CREATE TABLE ItemEntity (
    id TEXT NOT NULL PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    tags TEXT NOT NULL,
    status TEXT NOT NULL,
    category TEXT NOT NULL
);

getByCategory:
SELECT * FROM ItemEntity WHERE category = ?;

upsert:
INSERT OR REPLACE INTO ItemEntity (id, title, description, tags, status, category)
VALUES (?, ?, ?, ?, ?, ?);

observeAll:
SELECT * FROM ItemEntity;
```

### Ktor 네트워크 클라이언트 (KMP)

```kotlin
class ItemRemoteDataSource(private val client: HttpClient) {

    suspend fun fetchItems(category: String): List<ItemDto> {
        return client.get("api/items") {
            parameter("category", category)
        }.body()
    }
}

// 콘텐츠 협상(Content Negotiation)을 포함한 HttpClient 설정
val httpClient = HttpClient {
    install(ContentNegotiation) { json(Json { ignoreUnknownKeys = true }) }
    install(Logging) { level = LogLevel.HEADERS }
    defaultRequest { url("https://api.example.com/") }
}
```

## 의존성 주입 (Dependency Injection)

### Koin (KMP 친화적)

```kotlin
// 도메인 모듈
val domainModule = module {
    factory { GetItemsByCategoryUseCase(get()) }
    factory { ObserveUserProgressUseCase(get()) }
}

// 데이터 모듈
val dataModule = module {
    single<ItemRepository> { ItemRepositoryImpl(get(), get()) }
    single { ItemLocalDataSource(get()) }
    single { ItemRemoteDataSource(get()) }
}

// 프레젠테이션 모듈
val presentationModule = module {
    viewModelOf(::ItemListViewModel)
    viewModelOf(::DashboardViewModel)
}
```

### Hilt (Android 전용)

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    abstract fun bindItemRepository(impl: ItemRepositoryImpl): ItemRepository
}

@HiltViewModel
class ItemListViewModel @Inject constructor(
    private val getItems: GetItemsByCategoryUseCase
) : ViewModel()
```

## 에러 처리

### Result/Try 패턴

에러 전파를 위해 `Result<T>` 또는 커스텀 Sealed 타입을 사용하십시오.

```kotlin
sealed interface Try<out T> {
    data class Success<T>(val value: T) : Try<T>
    data class Failure(val error: AppError) : Try<Nothing>
}

sealed interface AppError {
    data class Network(val message: String) : AppError
    data class Database(val message: String) : AppError
    data object Unauthorized : AppError
}

// ViewModel에서 UI 상태로 매핑
viewModelScope.launch {
    when (val result = getItems(category)) {
        is Try.Success -> _state.update { it.copy(items = result.value, isLoading = false) }
        is Try.Failure -> _state.update { it.copy(error = result.error.toMessage(), isLoading = false) }
    }
}
```

## 컨벤션 플러그인 (Convention Plugins, Gradle)

KMP 프로젝트의 경우, 빌드 파일의 중복을 줄이기 위해 컨벤션 플러그인을 활용하십시오.

```kotlin
// build-logic/src/main/kotlin/kmp-library.gradle.kts
plugins {
    id("org.jetbrains.kotlin.multiplatform")
}

kotlin {
    androidTarget()
    iosX64(); iosArm64(); iosSimulatorArm64()
    sourceSets {
        commonMain.dependencies { /* 공통 의존성 */ }
        commonTest.dependencies { implementation(kotlin("test")) }
    }
}
```

모듈에서의 적용:

```kotlin
// domain/build.gradle.kts
plugins { id("kmp-library") }
```

## 지양해야 할 안티 패턴

- `domain` 레이어에서 Android 프레임워크 클래스 임포트 — 순수 Kotlin을 유지하십시오.
- 데이터베이스 엔티티나 DTO를 UI 레이어에 노출함 — 항상 도메인 모델로 매핑하십시오.
- 비즈니스 로직을 ViewModel에 작성함 — UseCase로 추출하십시오.
- `GlobalScope`나 비구조적 코루틴 사용 — `viewModelScope`나 구조적 동시성을 사용하십시오.
- 비대한 리포지토리 구현 — 세분화된 DataSource로 분리하십시오.
- 모듈 간 순환 의존성 — A가 B에 의존하면, B는 A에 의존해서는 안 됩니다.

## 참고 사항

UI 패턴에 대해서는 `compose-multiplatform-patterns` 스킬을 참고하십시오.
비동기 패턴에 대해서는 `kotlin-coroutines-flows` 스킬을 참고하십시오.
