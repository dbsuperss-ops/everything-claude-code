---
name: compose-multiplatform-patterns
description: KMP 프로젝트를 위한 Compose Multiplatform 및 Jetpack Compose 패턴 — 상태 관리, 내비게이션, 테마 설정, 성능 최적화 및 플랫폼별 UI 구현 지침.
origin: ECC
---

# Compose Multiplatform 패턴 (Patterns)

Compose Multiplatform 및 Jetpack Compose를 사용하여 Android, iOS, 데스크톱 및 웹에서 공용 UI를 구축하기 위한 패턴입니다. 상태 관리, 내비게이션, 테마 설정 및 성능 최적화를 다룹니다.

## 활성화 시점

- Compose UI(Jetpack Compose 또는 Compose Multiplatform)를 구축할 때
- ViewModel 및 Compose 상태를 사용하여 UI 상태를 관리할 때
- KMP 또는 Android 프로젝트에서 내비게이션을 구현할 때
- 재사용 가능한 컴포저블(Composables) 및 디자인 시스템을 설계할 때
- 리컴포지션(Recomposition) 및 렌더링 성능을 최적화할 때

## 상태 관리

### ViewModel + 단일 상태 객체

화면 상태를 위해 단일 데이터 클래스를 사용하십시오. 이를 `StateFlow`로 노출하고 Compose에서 수집(Collect)합니다:

```kotlin
data class ItemListState(
    val items: List<Item> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null,
    val searchQuery: String = ""
)

class ItemListViewModel(
    private val getItems: GetItemsUseCase
) : ViewModel() {
    private val _state = MutableStateFlow(ItemListState())
    val state: StateFlow<ItemListState> = _state.asStateFlow()

    fun onSearch(query: String) {
        _state.update { it.copy(searchQuery = query) }
        loadItems(query)
    }

    private fun loadItems(query: String) {
        viewModelScope.launch {
            _state.update { it.copy(isLoading = true) }
            getItems(query).fold(
                onSuccess = { items -> _state.update { it.copy(items = items, isLoading = false) } },
                onFailure = { e -> _state.update { it.copy(error = e.message, isLoading = false) } }
            )
        }
    }
}
```

### Compose에서 상태 수집하기

```kotlin
@Composable
fun ItemListScreen(viewModel: ItemListViewModel = koinViewModel()) {
    val state by viewModel.state.collectAsStateWithLifecycle()

    ItemListContent(
        state = state,
        onSearch = viewModel::onSearch
    )
}

@Composable
private fun ItemListContent(
    state: ItemListState,
    onSearch: (String) -> Unit
) {
    // 상태가 없는(Stateless) 컴포저블 — 프리뷰 및 테스트가 용이함
}
```

### 이벤트 싱크(Event Sink) 패턴

복잡한 화면의 경우, 여러 개의 콜백 람다 대신 sealed 인터페이스를 사용하여 이벤트를 정의하십시오:

```kotlin
sealed interface ItemListEvent {
    data class Search(val query: String) : ItemListEvent
    data class Delete(val itemId: String) : ItemListEvent
    data object Refresh : ItemListEvent
}

// ViewModel 내부
fun onEvent(event: ItemListEvent) {
    when (event) {
        is ItemListEvent.Search -> onSearch(event.query)
        is ItemListEvent.Delete -> deleteItem(event.itemId)
        is ItemListEvent.Refresh -> loadItems(_state.value.searchQuery)
    }
}

// 컴포저블 내부 — 여러 개 대신 단일 람다 사용
ItemListContent(
    state = state,
    onEvent = viewModel::onEvent
)
```

## 내비게이션 (Navigation)

### 타입 안전(Type-Safe) 내비게이션 (Compose Navigation 2.8 이상)

경로(Routes)를 `@Serializable` 객체로 정의하십시오:

```kotlin
@Serializable data object HomeRoute
@Serializable data class DetailRoute(val id: String)
@Serializable data object SettingsRoute

@Composable
fun AppNavHost(navController: NavHostController = rememberNavController()) {
    NavHost(navController, startDestination = HomeRoute) {
        composable<HomeRoute> {
            HomeScreen(onNavigateToDetail = { id -> navController.navigate(DetailRoute(id)) })
        }
        composable<DetailRoute> { backStackEntry ->
            val route = backStackEntry.toRoute<DetailRoute>()
            DetailScreen(id = route.id)
        }
        composable<SettingsRoute> { SettingsScreen() }
    }
}
```

### 다이얼로그 및 바텀 시트 내비게이션

명령형으로 보여주기/숨기기 기능을 구현하는 대신 `dialog()` 및 오버레이 패턴을 사용하십시오:

```kotlin
NavHost(navController, startDestination = HomeRoute) {
    composable<HomeRoute> { /* ... */ }
    dialog<ConfirmDeleteRoute> { backStackEntry ->
        val route = backStackEntry.toRoute<ConfirmDeleteRoute>()
        ConfirmDeleteDialog(
            itemId = route.itemId,
            onConfirm = { navController.popBackStack() },
            onDismiss = { navController.popBackStack() }
        )
    }
}
```

## 컴포저블 설계

### 슬롯 기반(Slot-Based) API

유연성을 위해 슬롯 파라미터를 사용하여 컴포저블을 설계하십시오:

```kotlin
@Composable
fun AppCard(
    modifier: Modifier = Modifier,
    header: @Composable () -> Unit = {},
    content: @Composable ColumnScope.() -> Unit,
    actions: @Composable RowScope.() -> Unit = {}
) {
    Card(modifier = modifier) {
        Column {
            header()
            Column(content = content)
            Row(horizontalArrangement = Arrangement.End, content = actions)
        }
    }
}
```

### Modifier 순서

Modifier의 순서는 매우 중요합니다. 다음 순서대로 적용하십시오:

```kotlin
Text(
    text = "Hello",
    modifier = Modifier
        .padding(16.dp)          // 1. 레이아웃 (여백, 크기)
        .clip(RoundedCornerShape(8.dp))  // 2. 형태 (모양)
        .background(Color.White) // 3. 그리기 (배경, 테두리)
        .clickable { }           // 4. 상호작용
)
```

## KMP 플랫폼별 UI

### 플랫폼 컴포저블을 위한 expect/actual

```kotlin
// commonMain
@Composable
expect fun PlatformStatusBar(darkIcons: Boolean)

// androidMain
@Composable
actual fun PlatformStatusBar(darkIcons: Boolean) {
    val systemUiController = rememberSystemUiController()
    SideEffect { systemUiController.setStatusBarColor(Color.Transparent, darkIcons) }
}

// iosMain
@Composable
actual fun PlatformStatusBar(darkIcons: Boolean) {
    // iOS는 UIKit 연동이나 Info.plist를 통해 이를 처리합니다.
}
```

## 성능 최적화

### 건너뛸 수 있는(Skippable) 리컴포지션을 위한 안정적 타입

모든 속성이 안정적일 경우 클래스를 `@Stable` 또는 `@Immutable`로 표시하십시오:

```kotlin
@Immutable
data class ItemUiModel(
    val id: String,
    val title: String,
    val description: String,
    val progress: Float
)
```

### `key()` 및 Lazy List의 올바른 사용

```kotlin
LazyColumn {
    items(
        items = items,
        key = { it.id }  // 안정적인 키(Stable keys)를 사용하면 아이템 재사용 및 애니메이션이 가능해집니다.
    ) { item ->
        ItemRow(item = item)
    }
}
```

### `derivedStateOf`를 사용한 읽기 지연

```kotlin
val listState = rememberLazyListState()
val showScrollToTop by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 5 }
}
```

### 리컴포지션 내 할당 방지

```kotlin
// 잘못된 예 — 리컴포지션마다 새로운 람다와 리스트 생성
items.filter { it.isActive }.forEach { ActiveItem(it, onClick = { handle(it) }) }

// 올바른 예 — 콜백이 올바른 행(row)에 유지되도록 각 아이템에 키를 지정
val activeItems = remember(items) { items.filter { it.isActive } }
activeItems.forEach { item ->
    key(item.id) {
        ActiveItem(item, onClick = { handle(item) })
    }
}
```

## 테마 설정 (Theming)

### Material 3 다이내믹 테마 (Dynamic Theming)

```kotlin
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            if (darkTheme) dynamicDarkColorScheme(LocalContext.current)
            else dynamicLightColorScheme(LocalContext.current)
        }
        darkTheme -> darkColorScheme()
        else -> lightColorScheme()
    }

    MaterialTheme(colorScheme = colorScheme, content = content)
}
```

## 피해야 할 안티 패턴

- 라이프사이클 관리 측면에서 `collectAsStateWithLifecycle`와 함께 사용하는 `MutableStateFlow`가 더 안전함에도 불구하고 ViewModel에서 `mutableStateOf`를 사용하는 것
- `NavController`를 컴포저블 깊숙이 전달하는 것 — 대신 람다 콜백을 전달하십시오.
- `@Composable` 함수 내부에서 무거운 연산을 수행하는 것 — ViewModel로 옮기거나 `remember {}`를 사용하십시오.
- `LaunchedEffect(Unit)`을 ViewModel 초기화 대용으로 사용하는 것 — 일부 설정에서는 구성 변경(Configuration change) 시 다시 실행될 수 있습니다.
- 컴포저블 파라미터에서 새로운 객체 인스턴스를 생성하는 것 — 불필요한 리컴포지션을 유발합니다.

## 참고 자료

- 모듈 구조 및 레이어링에 대해서는 `android-clean-architecture` 스킬을 참조하십시오.
- 코루틴 및 Flow 패턴에 대해서는 `kotlin-coroutines-flows` 스킬을 참조하십시오.
