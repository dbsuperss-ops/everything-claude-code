---
name: kotlin-coroutines-flows
description: Android 및 KMP를 위한 Kotlin Coroutines 및 Flow 패턴 — 구조화된 동시성, Flow 연산자, StateFlow, 에러 처리 및 테스트 가이드입니다.
origin: ECC
---

# Kotlin Coroutines & Flows

Android 및 Kotlin Multiplatform(KMP) 프로젝트에서 구조화된 동시성, Flow 기반 리액티브 스트림, 코루틴 테스트를 위한 패턴을 안내합니다.

## 활성화 시점

- Kotlin 코루틴을 사용하여 비동기 코드를 작성할 때
- 리액티브 데이터를 위해 Flow, StateFlow 또는 SharedFlow를 사용할 때
- 동시 작업(병렬 로딩, 디바운스, 재시도 등)을 처리할 때
- 코루틴 및 Flow를 테스트할 때
- 코루틴 범위(Scope) 및 취소(Cancellation)를 관리할 때

## 구조화된 동시성 (Structured Concurrency)

### 범위 계층 구조

```
Application
  └── viewModelScope (ViewModel)
        └── coroutineScope { } (구조적 자식)
              ├── async { } (동시 작업)
              └── async { } (동시 작업)
```

항상 구조화된 동시성을 사용하십시오. `GlobalScope`는 절대로 사용하지 마십시오.

```kotlin
// 나쁨
GlobalScope.launch { fetchData() }

// 좋음 — ViewModel 생명주기에 종속됨
viewModelScope.launch { fetchData() }

// 좋음 — Composable 생명주기에 종속됨
LaunchedEffect(key) { fetchData() }
```

### 병렬 분해 (Parallel Decomposition)

병렬 작업을 위해 `coroutineScope`와 `async`를 사용하십시오.

```kotlin
suspend fun loadDashboard(): Dashboard = coroutineScope {
    val items = async { itemRepository.getRecent() }
    val stats = async { statsRepository.getToday() }
    val profile = async { userRepository.getCurrent() }
    Dashboard(
        items = items.await(),
        stats = stats.await(),
        profile = profile.await()
    )
}
```

### SupervisorScope

자식 코루틴의 실패가 형제 코루틴에게 영향을 주지 않아야 할 때 사용하십시오.

```kotlin
suspend fun syncAll() = supervisorScope {
    launch { syncItems() }       // 여기서 실패해도 syncStats는 계속 진행됨
    launch { syncStats() }
    launch { syncSettings() }
}
```

## Flow 패턴

### StateFlow (UI 상태 관리용)

```kotlin
class DashboardViewModel(
    observeProgress: ObserveUserProgressUseCase
) : ViewModel() {
    val progress: StateFlow<UserProgress> = observeProgress()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000), // 마지막 구독자 이탈 후 5초간 유지
            initialValue = UserProgress.EMPTY
        )
}
```

### Flow 연산자 활용

```kotlin
// 검색 입력 디바운스
searchQuery
    .debounce(300)
    .distinctUntilChanged()
    .flatMapLatest { query -> repository.search(query) }
    .catch { emit(emptyList()) }
    .collect { results -> _state.update { it.copy(results = results) } }

// 지수 백오프를 이용한 재시도
fun fetchWithRetry(): Flow<Data> = flow { emit(api.fetch()) }
    .retryWhen { cause, attempt ->
        if (cause is IOException && attempt < 3) {
            delay(1000L * (1 shl attempt.toInt()))
            true
        } else {
            false
        }
    }
```

### SharedFlow (일회성 이벤트용)

스낵바 메시지나 내비게이션 이동과 같은 일회성 이벤트에 사용하십시오.

## 디스패처 (Dispatchers)

- `Dispatchers.Default`: CPU 집약적 작업
- `Dispatchers.IO`: 입출력(DB, 네트워크) 작업
- `Dispatchers.Main`: UI 스레드 작업

KMP 프로젝트에서는 모든 플랫폼에서 사용 가능한 `Default`와 `Main`을 주로 사용하십시오.

## 취소 (Cancellation)

코루틴은 협력적으로 취소되어야 합니다. 긴 루프 내에서는 `ensureActive()` 호출을 통해 취소 여부를 확인하십시오. `try-finally` 블록의 `finally` 절은 취소 시에도 자원 정리를 위해 항상 실행됩니다.

## 테스트

### Turbine을 이용한 Flow 테스트

```kotlin
@Test
fun `search updates item list`() = runTest {
    val viewModel = ItemListViewModel(...)

    viewModel.state.test {
        assertEquals(ItemListState(), awaitItem())  // 초기 상태 확인

        viewModel.onSearch("query")
        assertTrue(awaitItem().isLoading) // 로딩 상태 확인

        val loaded = awaitItem()
        assertFalse(loaded.isLoading)
        assertEquals(1, loaded.items.size)
    }
}
```

## 안티 패턴 (주의 사항)

- **`GlobalScope` 사용**: 코루틴 누수 및 취소 불가능의 원인입니다.
- **수정 가능한 컬렉션을 `MutableStateFlow`에 사용**: 항상 불변 복사본을 만들어 `update { it.copy(...) }` 방식으로 상태를 갱신하십시오.
- **`CancellationException` 캐치**: 취소가 정상적으로 전파되도록 이 예외는 다시 던지거나 잡지 마십시오.
