---
paths:
  - "**/*.kt"
  - "**/*.kts"
---
# Kotlin 보안 (Security)

> 이 문서는 [common/security.md](../common/security.md)의 규칙을 기반으로 Kotlin 및 Android/KMP 특화 내용을 확장합니다.

## 비밀 정보(Secrets) 관리

- 소스 코드 내에 API 키, 토큰 또는 자격 증명을 절대 하드코딩하지 마십시오.
- 로컬 개발용 비밀 정보는 `local.properties` (git-ignored) 파일을 사용하십시오.
- 릴리스 빌드 시에는 CI 비밀 정보로부터 생성된 `BuildConfig` 필드를 사용하십시오.
- 런타임 보안 저장을 위해 Android에서는 `EncryptedSharedPreferences`를, iOS에서는 Keychain을 사용하십시오.

```kotlin
// 잘못된 예
val apiKey = "sk-abc123..."

// 올바른 예 — BuildConfig 사용 (빌드 시 생성됨)
val apiKey = BuildConfig.API_KEY

// 올바른 예 — 런타임 보안 저장소 활용
val token = secureStorage.get("auth_token")
```

## 네트워크 보안

- HTTPS만 사용하십시오 — `network_security_config.xml`에서 cleartext 트래픽을 차단하도록 설정하십시오.
- 민감한 엔드포인트의 경우 OkHttp의 `CertificatePinner` 또는 Ktor의 동급 도구를 사용하여 인증서 핀닝(Certificate pinning)을 적용하십시오.
- 모든 HTTP 클라이언트에 타임아웃을 설정하십시오 — 무한 대기로 설정될 수 있는 기본값을 절대 그대로 사용하지 마십시오.
- 서버 응답을 사용하기 전에 반드시 검증하고 정제(Sanitize)하십시오.

```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false" />
</network-security-config>
```

## 입력값 검증

- 처리하거나 API로 전송하기 전에 모든 사용자 입력을 검증하십시오.
- Room 또는 SQLDelight 사용 시 매개변수화된 쿼리를 사용하십시오 — 절대로 SQL 문 내에 사용자 입력을 직접 결합하지 마십시오.
- 경로 탐색(Path traversal) 리스크를 방지하기 위해 사용자로부터 입력받은 파일 경로를 정제하십시오.

```kotlin
// 잘못된 예 — SQL 인젝션 취약
@Query("SELECT * FROM items WHERE name = '$input'")

// 올바른 예 — 매개변수화된 쿼리
@Query("SELECT * FROM items WHERE name = :input")
fun findByName(input: String): List<ItemEntity>
```

## 데이터 보호

- Android의 민감한 키-값 데이터에는 `EncryptedSharedPreferences`를 사용하십시오.
- `@Serializable` 사용 시 필드 이름을 명시적으로 지정하십시오 — 내부 속성 이름이 노출되지 않도록 주의하십시오.
- 더 이상 필요하지 않은 민감한 데이터는 메모리에서 즉시 삭제하십시오.
- 이름 변조(Name mangling) 방지를 위해 직렬화된 클래스에 대해 `@Keep` 어노테이션이나 ProGuard 규칙을 적용하십시오.

## 인증 (Authentication)

- 토큰은 일반 SharedPreferences가 아닌 보안 저장소에 보관하십시오.
- 401/403 응답 처리를 포함한 토큰 갱신(Refresh) 로직을 적절히 구현하십시오.
- 로그아웃 시 모든 인증 상태(토큰, 캐시된 사용자 데이터, 쿠키 등)를 삭제하십시오.
- 민감한 작업에 대해서는 생체 인증(`BiometricPrompt`) 사용을 고려하십시오.

## ProGuard / R8

- 모든 직렬화 모델(`@Serializable`, Gson, Moshi 등)에 대한 규칙을 유지하십시오.
- 리플렉션 기반 라이브러리(Koin, Retrofit 등)에 대한 규칙을 유지하십시오.
- 릴리스 빌드를 반드시 테스트하십시오 — 난독화 과정에서 직렬화 로직이 소리 없이 깨질 수 있습니다.

## WebView 보안

- 명시적으로 필요한 경우가 아니면 JavaScript를 비활성화하십시오: `settings.javaScriptEnabled = false`
- WebView에서 로드하기 전 URL의 유효성을 검증하십시오.
- 민감한 데이터에 접근하는 `@JavascriptInterface` 메서드를 외부에 노출하지 마십시오.
- 내비게이션 제어를 위해 `WebViewClient.shouldOverrideUrlLoading()`을 활용하십시오.
