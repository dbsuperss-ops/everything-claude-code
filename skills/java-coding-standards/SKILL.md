---
name: java-coding-standards
description: Spring Boot 서비스를 위한 Java 코딩 표준 가이드입니다. 명명 규칙, 불변성, Optional 사용법, 스트림, 예외 처리, 제네릭 및 프로젝트 구조를 다룹니다.
origin: ECC
---

# Java 코딩 표준 (Java Coding Standards)

Spring Boot 서비스에서 읽기 쉽고 유지보수가 용이한 Java(17+) 코드를 작성하기 위한 표준입니다.

## 활성화 시점

- Spring Boot 프로젝트에서 Java 코드를 작성하거나 검토할 때
- 명명 규칙, 불변성 공식, 또는 예외 처리 규칙을 적용할 때
- 레코드(Records), 실드 클래스(Sealed classes), 패턴 매칭 등 Java 17+ 기능을 사용할 때
- `Optional`, 스트림, 또는 제네릭 사용법을 검토할 때
- 패키지 구조 및 프로젝트 레이아웃을 구성할 때

## 핵심 원칙

- 기교(Cleverness)보다 명확성(Clarity)을 지향하십시오.
- 기본적으로 불변성(Immutable)을 유지하고 공유된 가변 상태를 최소화하십시오.
- 의미 있는 예외와 함께 "Fail Fast" 원칙을 지키십시오.
- 일관된 명명 규칙과 패키지 구조를 유지하십시오.

## 명명 규칙 (Naming)

- **클래스/레코드**: PascalCase (예: `MarketService`, `Money`)
- **메서드/필드**: camelCase (예: `marketRepository.findBySlug`)
- **상수**: UPPER_SNAKE_CASE (예: `MAX_PAGE_SIZE`)

## 불변성 (Immutability)

- 레코드(Records)와 `final` 필드를 적극 활용하십시오.
- Getter만 제공하고 Setter는 가급적 만들지 마십시오.

## Optional 사용법

- `find*` 계열의 메서드는 `Optional`을 반환하십시오.
- `.get()` 대신 `.map()`, `.flatMap()`, `.orElseThrow()` 등을 사용하여 안전하게 처리하십시오.

## 스트림(Streams) 최선 관행

- 변환(Transformation) 작업에 스트림을 사용하되, 파이프라인을 너무 길게 만들지 마십시오.
- 복잡하게 중첩된 스트림은 가독성을 위해 루프로 교체하는 것을 고려하십시오.

## 예외 처리 (Exceptions)

- 도메인 에러에는 언체크(Unchecked) 예외를 사용하고, 기술적인 예외는 문맥과 함께 감싸서 다시 던지십시오.
- 도메인 특화 예외(예: `MarketNotFoundException`)를 생성하십시오.
- 무분별한 `catch (Exception ex)`를 피하십시오.

## 프로젝트 구조 (Maven/Gradle)

- `controller`, `service`, `repository`, `domain`, `dto`, `config`, `util` 등으로 패키지를 분리하십시오.

## 로깅 (Logging)

- `SLF4J`를 사용하여 로그를 기록하십시오. (예: `log.info("fetch_market slug={}", slug);`)

## 기타

- **성능**: 마이크로 최적화보다 유지보수성을 우선하십시오. 필요성이 입증된 경우에만 최적화하십시오.
- **테스트**: JUnit 5와 AssertJ를 사용하여 읽기 쉬운 단언문(Assertion)을 작성하십시오. 모킹에는 Mockito를 사용하십시오.

**기억하십시오**: 코드는 의도가 명확해야 하며, 타입 안전성을 보장하고 관찰 가능해야 합니다.
    
