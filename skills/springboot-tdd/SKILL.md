---
name: springboot-tdd
description: JUnit 5, Mockito, MockMvc, Testcontainers 및 JaCoCo를 사용한 Spring Boot 테스트 주도 개발(TDD) 가이드입니다. 기능 추가, 버그 수정 또는 리팩토링 시 사용하십시오.
origin: ECC
---

# Spring Boot TDD 워크플로우 (Spring Boot TDD Workflow)

단위 테스트와 통합 테스트를 포함하여 80% 이상의 커버리지를 목표로 하는 Spring Boot 서비스를 위한 TDD 가이드입니다.

## 활성화 시점

- 새로운 기능 또는 엔드포인트를 개발할 때
- 버그 수정 또는 리팩토링 시
- 데이터 접근 로직이나 보안 규칙을 추가할 때

## 워크플로우 (Workflow)

1. **테스트 먼저 작성**: 의도한 동작에 대해 실패하는 테스트를 작성하십시오.
2. **최소 구현**: 테스트를 통과시키기 위한 최소한의 코드를 작성하십시오.
3. **리팩토링**: 테스트가 통과(Green)하는 상태를 유지하며 코드를 개선하십시오.
4. **커버리지 강제**: JaCoCo 등을 사용하여 필요한 커버리지를 확보하십시오.

## 단위 테스트 (JUnit 5 + Mockito)

- `@ExtendWith(MockitoExtension.class)`와 `@Mock`, `@InjectMocks`를 사용하십시오.
- **Arrange-Act-Assert** 패턴을 따르십시오.
- 부분 모킹(Partial mocks)을 피하고 명시적인 스터빙(Stubbing)을 선호하십시오.
- 다양한 입력 케이스에는 `@ParameterizedTest`를 활용하십시오.

## 웹 계층 테스트 (MockMvc)

- `@WebMvcTest`를 사용하여 특정 컨트롤러와 관련된 빈만 로드하십시오.
- `MockMvc`를 사용하여 HTTP 요청을 시뮬레이션하고 응답 상태 및 JSON 경로(`jsonPath`)를 검증하십시오.

## 통합 및 영속성 테스트

- **통합 테스트**: `@SpringBootTest`와 `@AutoConfigureMockMvc`를 사용하여 전체 컨텍스트를 테스트하십시오.
- **영속성 테스트**: `@DataJpaTest`를 사용하여 리포지토리 레이어를 테스트하십시오.
- **Testcontainers**: 운영 환경과 유사한 DB(Postgres, Redis 등) 환경을 위해 **Testcontainers**를 적극 활용하십시오.

## 주요 도구 및 최선 관행

- **AssertJ**: 가독성을 위해 `assertThat` 기반의 유창한(Fluent) 단언문을 사용하십시오.
- **테스트 데이터 빌더**: 테스트용 객체 생성을 위해 빌더 패턴을 활용하여 중복을 줄이십시오.
- **속도와 결정론**: 테스트는 빠르고, 독립적이며, 항상 같은 결과를 내야 합니다. 내부 구현 상세가 아닌 **동작(Behavior)**을 테스트하십시오.

**기억하십시오**: 테스트는 코드의 품질뿐만 아니라 설계의 결함을 조기에 발견할 수 있는 가장 강력한 도구입니다.
    
