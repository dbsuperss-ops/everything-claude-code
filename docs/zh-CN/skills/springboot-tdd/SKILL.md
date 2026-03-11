---
name: springboot-tdd
description: JUnit 5, Mockito, MockMvc, Testcontainers 및 JaCoCo를 사용한 Spring Boot 테스트 주도 개발(TDD) 가이드입니다. 기능 추가, 버그 수정 또는 리팩토링 시 활용하십시오.
origin: ECC
---

# Spring Boot TDD 워크플로우

Spring Boot 서비스의 전체 테스트 커버리지 80% 이상(단위 + 통합)을 달성하기 위한 TDD 지침입니다.

## 적용 시점

* 새로운 기능 또는 API 엔드포인트를 개발할 때
* 버그 수정 또는 리팩토링을 수행할 때
* 데이터 액세스 로직이나 보안 규칙을 추가할 때

## 워크플로우 단계

1. **테스트 우선 작성**: 요구 사항에 맞는 테스트를 먼저 작성합니다 (이 시점엔 실패해야 함).
2. **코드 구현**: 테스트를 통과하기 위한 최소한의 코드를 작성합니다.
3. **리팩토링**: 테스트 성공을 유지하면서 코드를 개선하고 정리합니다.
4. **커버리지 확인**: JaCoCo 등을 통해 목표 커버리지(80%+) 달성 여부를 확인합니다.

## 단위 테스트 (JUnit 5 + Mockito)

```java
@ExtendWith(MockitoExtension.class)
class MarketServiceTest {
  @Mock MarketRepository repo;
  @InjectMocks MarketService service;

  @Test
  void createsMarket() {
    // Arrange (준비)
    CreateMarketRequest req = new CreateMarketRequest("이름", "설명", Instant.now(), List.of("카테고리"));
    when(repo.save(any())).thenAnswer(inv -> inv.getArgument(0));

    // Act (실행)
    Market result = service.create(req);

    // Assert (검증)
    assertThat(result.name()).isEqualTo("이름");
    verify(repo).save(any());
  }
}
```

## 웹 계층 테스트 (MockMvc)

`@WebMvcTest`를 사용하여 컨트롤러와 MVC 설정만 로드하고 서비스는 모킹합니다.

```java
@WebMvcTest(MarketController.class)
class MarketControllerTest {
  @Autowired MockMvc mockMvc;
  @MockBean MarketService marketService;

  @Test
  void returnsMarkets() throws Exception {
    when(marketService.list(any())).thenReturn(Page.empty());

    mockMvc.perform(get("/api/markets"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.content").isArray());
  }
}
```

## 통합 테스트 및 환경 구성

* **SpringBootTest**: 실제 스프링 컨텍스트를 로드하여 전체 흐름을 테스트합니다.
* **Testcontainers**: Postgres, Redis 등 실제 인프라를 컨테이너로 띄워 운영 환경과 유사한 환경에서 테스트하십시오.
* **AssertJ**: 가독성 높은 단언문(`assertThat`)을 사용하십시오.

## 실행 명령어

* **Maven**: `mvn test` 또는 `mvn verify`
* **Gradle**: `./gradlew test jacocoTestReport`

**핵심**: 테스트는 구현 상세 내용이 아닌 '비즈니스 행위'를 검증해야 합니다. 빠르고, 격리되며, 결정론적인(언제든 결과가 같은) 테스트를 작성하십시오.
