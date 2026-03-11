---
name: springboot-patterns
description: Spring Boot 아키텍처 패턴, REST API 설계, 계층형 서비스, 데이터 액세스, 캐싱, 비동기 처리 및 로깅 가이드입니다. Java Spring Boot 백엔드 개발 시 활용하십시오.
origin: ECC
---

# Spring Boot 개발 패턴

확장 가능하고 프로덕션 급 서비스를 구축하기 위한 Spring Boot 아키텍처 및 API 패턴입니다.

## 적용 시점

* Spring MVC 또는 WebFlux를 사용하여 REST API를 구축할 때
* Controller → Service → Repository 계층 구조를 설계할 때
* Spring Data JPA, 캐싱(Cache), 비동기(Async) 처리를 설정할 때
* 유효성 검증(Validation), 예외 처리(Exception Handling), 페이지네이션을 추가할 때
* 개발/스테이징/운영 환경별 프로파일(Profiles)을 설정할 때
* Spring Events 또는 Kafka를 활용한 이벤트 기반 패턴을 구현할 때

## REST API 구조

```java
@RestController
@RequestMapping("/api/markets")
@Validated
class MarketController {
  private final MarketService marketService;

  MarketController(MarketService marketService) {
    this.marketService = marketService;
  }

  @GetMapping
  ResponseEntity<Page<MarketResponse>> list(
      @RequestParam(defaultValue = "0") int page,
      @RequestParam(defaultValue = "20") int size) {
    Page<Market> markets = marketService.list(PageRequest.of(page, size));
    return ResponseEntity.ok(markets.map(MarketResponse::from));
  }

  @PostMapping
  ResponseEntity<MarketResponse> create(@Valid @RequestBody CreateMarketRequest request) {
    Market market = marketService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(MarketResponse.from(market));
  }
}
```

## 서비스 계층 (트랜잭션 관리)

```java
@Service
public class MarketService {
  private final MarketRepository repo;

  public MarketService(MarketRepository repo) {
    this.repo = repo;
  }

  @Transactional
  public Market create(CreateMarketRequest request) {
    MarketEntity entity = MarketEntity.from(request);
    MarketEntity saved = repo.save(entity);
    return Market.from(saved);
  }

  @Transactional(readOnly = true)
  public Page<Market> list(Pageable pageable) {
    return repo.findAll(pageable).map(Market::from);
  }
}
```

## 예외 처리 (Global Exception Handler)

```java
@ControllerAdvice
class GlobalExceptionHandler {
  @ExceptionHandler(MethodArgumentNotValidException.class)
  ResponseEntity<ApiError> handleValidation(MethodArgumentNotValidException ex) {
    String message = ex.getBindingResult().getFieldErrors().stream()
        .map(e -> e.getField() + ": " + e.getDefaultMessage())
        .collect(Collectors.joining(", "));
    return ResponseEntity.badRequest().body(ApiError.validation(message));
  }

  @ExceptionHandler(Exception.class)
  ResponseEntity<ApiError> handleGeneric(Exception ex) {
    // 상세 로그 기록 (스택 트레이스 포함)
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(ApiError.of("내부 서버 오류가 발생했습니다."));
  }
}
```

## 캐싱 및 비동기 처리

* **캐싱**: `@EnableCaching`을 설정한 후, 반복 조회되는 데이터에 `@Cacheable`을 적용하십시오.
* **비동기**: `@EnableAsync`를 설정한 후, 이메일 발송 등 시간이 소요되는 작업에 `@Async`를 적용하십시오.

## 로깅 (SLF4J)

```java
@Service
public class ReportService {
  private static final Logger log = LoggerFactory.getLogger(ReportService.class);

  public Report generate(Long marketId) {
    log.info("리포트 생성 시작 marketId={}", marketId);
    try {
      // 로직 수행
    } catch (Exception ex) {
      log.error("리포트 생성 실패 marketId={}", marketId, ex);
      throw ex;
    }
    return new Report();
  }
}
```

## 핵심 규칙 및 권장 사항

1. **생성자 주입 권장**: 필드 주입(`@Autowired`) 대신 생성자 주입을 사용하여 테스트 용이성과 불변성을 확보하십시오.
2. **DTO 사용**: 엔티티(Entity)를 직접 API 응답으로 노출하지 말고, DTO(Record 권장)를 사용하십시오.
3. **읽기 전용 트랜잭션**: 단순 조회 작업에는 `@Transactional(readOnly = true)`를 사용하여 성능을 최적화하십시오.
4. **Validation**: Pydantic(백엔드) 대응 개념인 `@Valid`, `@NotBlank`, `@Min` 등을 적극 활용하십시오.
5. **구조화된 로그**: 인덱싱이 용이하도록 JSON 형식이나 명확한 키-값 구조로 로그를 남기십시오.

**핵심**: 컨트롤러는 가볍게, 서비스는 비즈니스 로직에 집중하며, 예외 처리는 한곳에서 집중 관리하십시오. 유지보수와 테스트가 쉬운 코드가 좋은 Spring Boot 코드입니다.
