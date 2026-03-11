---
name: java-coding-standards
description: Spring Boot 서비스를 위한 Java 코딩 표준 가이드입니다. 명명 규칙, 불변성, Optional 사용법, 스트림, 예외 처리, 제네릭 및 프로젝트 구조를 다룹니다.
origin: ECC
---

# Java 코딩 표준

Spring Boot 서비스에서 가독성 있고 유지보수가 용이한 Java (17+) 코드를 작성하기 위한 가이드라인입니다.

## 적용 시점

* Spring Boot 프로젝트에서 Java 코드를 작성하거나 리뷰할 때
* 명명 규칙, 불변성 또는 예외 처리 규약을 강제해야 할 때
* 레코드(Records), 봉인된 클래스(Sealed classes) 또는 패턴 매칭(Java 17+)을 사용할 때
* Optional, 스트림(Streams) 또는 제네릭(Generics) 사용을 검토할 때
* 패키지 및 프로젝트 레이아웃을 구성할 때

## 핵심 원칙

* 기교보다는 명확성(Clarity)을 우선시하십시오.
* 기본적으로 불변성(Immutability)을 유지하고, 공유 가변 상태를 최소화하십시오.
* 의미 있는 예외와 함께 빠르게 실패(Fail-fast)하십시오.
* 일관된 명명 규칙과 패키지 구조를 유지하십시오.

## 명명 규칙 (Naming)

```java
// ✅ 클래스 & 레코드: PascalCase
public class MarketService {}
public record Money(BigDecimal amount, Currency currency) {}

// ✅ 메서드 & 필드: camelCase
private final MarketRepository marketRepository;
public Market findBySlug(String slug) {}

// ✅ 상수: UPPER_SNAKE_CASE
private static final int MAX_PAGE_SIZE = 100;
```

## 불변성 (Immutability)

```java
// ✅ 레코드와 final 필드를 지향하십시오.
public record MarketDto(Long id, String name, MarketStatus status) {}

public class Market {
  private final Long id;
  private final String name;
  // Getter만 제공하고 Setter는 지양하십시오.
}
```

## Optional 사용법

```java
// ✅ find* 메서드에서 Optional을 반환하십시오.
Optional<Market> market = marketRepository.findBySlug(slug);

// ✅ get() 대신 map/flatMap과 orElseThrow를 사용하십시오.
return market
    .map(MarketResponse::from)
    .orElseThrow(() -> new EntityNotFoundException("마켓을 찾을 수 없습니다: " + slug));
```

## 스트림 (Streams) 베스트 프랙티스

```java
// ✅ 데이터 변환에 스트림을 사용하고, 파이프라인을 짧게 유지하십시오.
List<String> names = markets.stream()
    .map(Market::name)
    .filter(Objects::nonNull)
    .toList();

// ❌ 복잡하게 중첩된 스트림은 피하십시오. 명확성을 위해 단순 반복문(Loop)이 더 나을 수 있습니다.
```

## 예외 처리 (Exceptions)

* 도메인 에러에는 언체크 예외(Unchecked Exception)를 사용하십시오. 기술적 예외를 래핑할 때는 컨텍스트를 제공하십시오.
* 도메인 특화 예외를 생성하십시오 (예: `MarketNotFoundException`).
* 중앙 집중식으로 다시 던지거나 로깅하는 경우가 아니라면 광범위한 `catch (Exception ex)`를 피하십시오.

```java
throw new MarketNotFoundException(slug);
```

## 제네릭 및 타입 안정성

* 원시 타입(Raw types)을 피하고 제네릭 파라미터를 선언하십시오.
* 재사용 가능한 유틸리티 클래스에서는 가급적 한정적 와일드카드(Bounded generics)를 사용하십시오.

```java
public <T extends Identifiable> Map<Long, T> indexById(Collection<T> items) { ... }
```

## 프로젝트 구조 (Maven/Gradle)

```
src/main/java/com/example/app/
  config/         # 설정 레이어
  controller/     # web/api 엔드포인트
  service/        # 비즈니스 로직
  repository/     # 데이터 접근
  domain/         # 엔티티 및 핵심 도메인 모델
  dto/            # 데이터 전송 객체
  util/           # 유틸리티 함수
src/main/resources/
  application.yml
src/test/java/... (main 구조와 동일하게 유지)
```

## 포맷팅 및 스타일

* 프로젝트 표준에 따라 2개 또는 4개의 공백을 일관되게 사용하십시오.
* 파일당 하나의 공공 최상위 타입(Public top-level type)만 유지하십시오.
* 메서드는 짧고 집중적으로 유지하고, 필요시 헬퍼 메서드를 추출하십시오.
* 멤버 순서: 상수 -> 필드 -> 생성자 -> 공공 메서드 -> 보호/사설 메서드

## 피해야 할 코드 취약점 (Code Smells)

* 긴 파라미터 리스트 → DTO나 빌더(Builder) 패턴 사용
* 깊은 중첩 → Early Return(빠른 반환) 활용
* 매직 넘버 → 명명된 상수 사용
* 정적 가변 상태 → 의존성 주입(DI) 지향
* 빈 catch 블록 → 로그를 남기거나 처리하거나 다시 던지기

## 로깅 (Logging)

```java
private static final Logger log = LoggerFactory.getLogger(MarketService.class);
log.info("fetch_market slug={}", slug);
log.error("failed_fetch_market slug={}", slug, ex);
```

## Null 처리

* 불가피한 경우에만 `@Nullable`을 허용하고, 그 외에는 `@NonNull`을 사용하십시오.
* 입력값에 대해 Bean Validation (`@NotNull`, `@NotBlank` 등)을 사용하십시오.

## 테스트 원칙

* 유창한 단언(Fluent assertions)을 위해 JUnit 5 + AssertJ를 사용하십시오.
* 모킹(Mocking)에는 Mockito를 사용하되, 가급적 부분 모킹(Partial mocking)은 피하십시오.
* 결정론적인 테스트를 지향하십시오 (Thread.sleep() 등 숨겨진 대기 지양).

**핵심**: 코드의 의도를 명확히 하고 타입 안정성과 가관측성(Observability)을 확보하십시오. 입증된 필요성이 없는 한 미세 최적화보다 유지보수성을 우선시하십시오.
