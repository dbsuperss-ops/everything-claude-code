---
name: springboot-patterns
description: Spring Boot 아키텍처 패턴, REST API 설계, 계층형 서비스, 데이터 액세스, 캐싱, 비동기 처리 및 로깅 가이드입니다. Java Spring Boot 백엔드 작업에 사용하십시오.
origin: ECC
---

# Spring Boot 개발 패턴 (Spring Boot Development Patterns)

확장 가능하고 운영 환경에 적합한 서비스를 구축하기 위한 Spring Boot 아키텍처 및 API 패턴을 안내합니다.

## 활성화 시점

- Spring MVC 또는 WebFlux를 사용하여 REST API를 구축할 때
- Controller → Service → Repository 계층 구조를 설계할 때
- Spring Data JPA, 캐싱, 또는 비동기 처리를 설정할 때
- 데이터 검증(Validation), 예외 처리, 또는 페이지네이션을 추가할 때
- 개발/스테이징/운영 환경별 프로파일(Profiles)을 설정할 때
- Spring Events나 Kafka를 사용한 이벤트 기반 패턴을 구현할 때

## REST API 구조

- `@RestController`와 `@RequestMapping`을 사용하여 엔드포인트를 정의하십시오.
- `@Valid`와 `@RequestBody`를 조합하여 입력 데이터를 검증하십시오.
- `ResponseEntity`를 반환하여 상태 코드와 본문을 명시적으로 제어하십시오.

## 서비스 계층 및 트랜잭션

- 비즈니스 로직은 `@Service` 계층에 작성하십시오.
- 데이터 변경 작업에는 `@Transactional`을 사용하고, 단순 조회에는 `@Transactional(readOnly = true)`를 사용하여 최적화하십시오.

## DTO 및 데이터 검증

- Java 14+의 `record`를 사용하여 불변 DTO를 정의하십시오.
- `@NotBlank`, `@Size`, `@NotNull`, `@FutureOrPresent` 등 Bean Validation 어노테이션을 활용하십시오.

## 예외 처리 (Global Exception Handling)

- `@ControllerAdvice` 또는 `@RestControllerAdvice`를 사용하여 애플리케이션 전역의 예외를 공통된 형식(예: `ApiError`)으로 처리하십시오.

## 캐싱 및 비동기 처리

- `@EnableCaching`과 `@Cacheable`, `@CacheEvict`를 사용하여 반복적인 조회 성능을 개선하십시오.
- `@EnableAsync`와 `@Async`를 사용하여 이메일 발송 등 시간이 걸리는 작업을 별도 스레드에서 처리하십시오.

## 로깅 및 미들웨어

- SLF4J(`Logger`)를 사용하여 정보와 에러를 기록하십시오.
- `OncePerRequestFilter`를 상속받아 요청 로깅, 속도 제한(Rate Limiting) 등의 공통 로직을 처리하십시오.

## 운영 환경 고려 사항

- **의존성 주입**: 필드 주입보다 생성자 주입을 권장합니다.
- **오브저버빌리티(Observability)**: Micrometer, Prometheus 등을 사용하여 메트릭과 트레이싱을 설정하십시오.
- **안정성**: 외부 API 호출 시 재시도(Retry) 로직과 타임아웃을 설정하여 장애 전파를 방지하십시오.

**기억하십시오**: 컨트롤러는 얇게(Thin), 서비스는 비즈니스 로직에 집중(Focused), 리포지토리는 단순하게 유지하며, 모든 에러는 중앙에서 일관되게 처리하십시오. 유지보수성과 테스트 용이성을 최우선으로 고려하십시오.
    
