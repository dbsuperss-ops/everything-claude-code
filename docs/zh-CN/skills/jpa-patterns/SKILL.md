---
name: jpa-patterns
description: Spring Boot에서의 JPA/Hibernate 패턴 가이드입니다. 엔티티 설계, 연관 관계 처리, 쿼리 최적화, 트랜잭션 관리, 감사(Auditing), 인덱싱, 페이지네이션 및 커넥션 풀 설정을 다룹니다.
origin: ECC
---

# JPA/Hibernate 패턴

Spring Boot에서의 데이터 모델링, 리포지토리(Repository) 구현 및 성능 최적화를 위한 가이드입니다.

## 적용 시점

* JPA 엔티티와 테이블 매핑을 설계할 때
* 연관 관계(@OneToMany, @ManyToOne, @ManyToMany)를 정의할 때
* 쿼리 성능을 최적화할 때 (N+1 문제 방지, 페치 전략, 프로젝션 활용)
* 트랜잭션, 감사(Auditing) 또는 소프트 삭제(Soft delete)를 설정할 때
* 페이지네이션, 정렬 또는 사용자 정의 리포지토리 메서드를 구현할 때
* 커넥션 풀(HikariCP)이나 2차 캐시를 튜닝할 때

## 엔티티 설계 (Entity Design)

```java
@Entity
@Table(name = "markets", indexes = {
  @Index(name = "idx_markets_slug", columnList = "slug", unique = true)
})
@EntityListeners(AuditingEntityListener.class)
public class MarketEntity {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(nullable = false, length = 200)
  private String name;

  @Column(nullable = false, unique = true, length = 120)
  private String slug;

  @Enumerated(EnumType.STRING)
  private MarketStatus status = MarketStatus.ACTIVE;

  @CreatedDate private Instant createdAt;
  @LastModifiedDate private Instant updatedAt;
}
```

감사(Auditing) 활성화 설정:

```java
@Configuration
@EnableJpaAuditing
class JpaConfig {}
```

## 연관 관계 및 N+1 방지

```java
@OneToMany(mappedBy = "market", cascade = CascadeType.ALL, orphanRemoval = true)
private List<PositionEntity> positions = new ArrayList<>();
```

* 기본적으로 지연 로딩(LAZY)을 사용하십시오. 필요한 경우에만 쿼리에서 `JOIN FETCH`를 사용합니다.
* 컬렉션에 `EAGER` 전략을 사용하는 것을 피하십시오. 조회 경로에는 DTO 프로젝션을 활용하십시오.

```java
@Query("select m from MarketEntity m left join fetch m.positions where m.id = :id")
Optional<MarketEntity> findWithPositions(@Param("id") Long id);
```

## 리포지토리 패턴 (Repository)

```java
public interface MarketRepository extends JpaRepository<MarketEntity, Long> {
  Optional<MarketEntity> findBySlug(String slug);

  @Query("select m from MarketEntity m where m.status = :status")
  Page<MarketEntity> findByStatus(@Param("status") MarketStatus status, Pageable pageable);
}
```

* 가벼운 조회를 위해 인터페이스 기반 프로젝션을 사용하십시오:

```java
public interface MarketSummary {
  Long getId();
  String getName();
  MarketStatus getStatus();
}
Page<MarketSummary> findAllBy(Pageable pageable);
```

## 트랜잭션 관리 (Transactions)

* 서비스 메서드에 `@Transactional` 어노테이션을 사용하십시오.
* 단순 조회 경로에는 최적화를 위해 `@Transactional(readOnly = true)`를 사용하십시오.
* 전파(Propagation) 동작을 신중히 선택하고, 장시간 실행되는 트랜잭션은 피하십시오.

```java
@Transactional
public Market updateStatus(Long id, MarketStatus status) {
  MarketEntity entity = repo.findById(id)
      .orElseThrow(() -> new EntityNotFoundException("마켓을 찾을 수 없습니다: " + id));
  entity.setStatus(status);
  return Market.from(entity);
}
```

## 페이지네이션 (Pagination)

```java
PageRequest page = PageRequest.of(pageNumber, pageSize, Sort.by("createdAt").descending());
Page<MarketEntity> markets = repo.findByStatus(MarketStatus.ACTIVE, page);
```

커서 기반 페이지네이션의 경우, JPQL에서 정렬과 함께 `id > :lastId` 조건을 포함하십시오.

## 인덱스 및 성능

* 자주 사용되는 필터(`status`, `slug`, 외래 키)에 인덱스를 추가하십시오.
* 쿼리 패턴과 일치하는 복합 인덱스(`status, created_at`)를 사용하십시오.
* `select *`를 피하고 필요한 컬럼만 프로젝션하십시오.
* 대량 쓰기 작업 시 `saveAll`과 `hibernate.jdbc.batch_size` 설정을 활용하십시오.

## 커넥션 풀 (HikariCP)

권장 속성 예시:

```properties
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.validation-timeout=5000
```

PostgreSQL LOB 처리를 위한 추가 설정:

```properties
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```

## 캐싱 전략

* 1차 캐시는 `EntityManager` 단위입니다. 트랜잭션 간에 엔티티를 유지하지 마십시오.
* 조회가 빈번한 엔티티에 대해서는 2차 캐시를 신중하게 고려하십시오. 만료(Eviction) 정책을 검증해야 합니다.

## 데이터베이스 마이그레이션

* Flyway나 Liquibase를 사용하십시오. 운영 환경에서 Hibernate의 자동 DDL 생성(`ddl-auto`)에 의존해서는 절대 안 됩니다.
* 마이그레이션은 멱등성(Idempotent)을 유지하고 점진적으로(Additive) 구성하십시오. 계획 없는 컬럼 삭제는 피하십시오.

## 테스트 데이터 액세스

* 운영 환경과 유사한 환경을 위해 Testcontainers를 사용한 `@DataJpaTest`를 권장합니다.
* SQL 효율성을 확인하려면 로그 설정을 활용하십시오: `logging.level.org.hibernate.SQL=DEBUG` 및 `logging.level.org.hibernate.orm.jdbc.bind=TRACE`.

**핵심**: 엔티티는 가볍게, 쿼리는 타겟 지향적으로, 트랜잭션은 짧게 유지하십시오. 페치 전략과 프로젝션으로 N+1 문제를 예방하고, 읽기/쓰기 패턴에 최적화된 인덱스를 설계하십시오.
