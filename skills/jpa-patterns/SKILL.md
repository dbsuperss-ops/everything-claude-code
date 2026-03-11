---
name: jpa-patterns
description: Spring Boot에서의 엔티티 설계, 연관관계, 쿼리 최적화, 트랜잭션, 감사(Auditing), 인덱싱 및 페이지네이션을 위한 JPA/Hibernate 패턴 가이드입니다.
origin: ECC
---

# JPA/Hibernate 패턴 (JPA/Hibernate Patterns)

Spring Boot에서 데이터 모델링, 리포지토리 구성 및 성능 튜닝을 위해 이 패턴을 사용하십시오.

## 활성화 시점

- JPA 엔티티 설계 및 테이블 매핑 시
- 연관관계(@OneToMany, @ManyToOne, @ManyToMany)를 정의할 때
- 쿼리 최적화(N+1 방지, 페치 전략, 프로젝션)가 필요할 때
- 트랜잭션, 감사(Auditing), 또는 소프트 딜리트(Soft delete) 설정 시
- 페이징, 정렬, 또는 커스텀 리포지토리 메서드 구현 시
- 커넥션 풀(HikariCP) 튜닝이나 2차 캐시 설정 시

## 엔티티 설계 (Entity Design)

- `@Entity`, `@Table`(인덱스 포함), `@EntityListeners(AuditingEntityListener.class)`를 기본으로 사용하십시오.
- `@CreatedDate`, `@LastModifiedDate`를 사용하여 생성/수정 시간을 자동 관리하십시오.
- `@EnableJpaAuditing` 설정을 잊지 마십시오.

## 연관관계 및 N+1 문제 방지

- 기본적으로 **지연 로딩(Lazy Loading)**을 사용하십시오.
- 컬렉션에 `EAGER`를 반환하지 마십시오. 성능 저하의 주원인입니다.
- 필요한 경우 JPQL에서 `JOIN FETCH`를 사용하거나 DTO 프로젝션을 활용하여 N+1 문제를 방지하십시오.

## 리포지토리 패턴 (Repository Patterns)

- `JpaRepository`를 상속받아 표준 메서드를 활용하십시오.
- 가벼운 조회 작업을 위해 **인터페이스 기반 프로젝션**을 사용하여 필요한 컬럼만 조회하십시오.

## 트랜잭션 (Transactions)

- 서비스 메서드에 `@Transactional`을 선언하십시오.
- 단순 조회 메서드에는 `@Transactional(readOnly = true)`를 사용하여 최적화하십시오.
- 트랜잭션의 범위를 너무 길게 잡지 마십시오.

## 커넥션 풀 및 인덱싱

- **인덱스**: 필터링(`status`, `slug`)이나 외래 키에 인덱스를 추가하십시오. 복합 인덱스는 쿼리 패턴에 맞춰 설계하십시오.
- **HikariCP**: 최대 풀 사이즈, 타임아웃 등 세부 속성을 환경에 맞춰 설정하십시오.
- **배치 처리**: 대량 쓰기 시 `saveAll`을 사용하고 `hibernate.jdbc.batch_size` 설정을 고려하십시오.

## 마이그레이션

- 운영 환경에서 Hibernate의 자동 DDL(`ddl-auto: update` 등)을 절대 사용하지 마십시오.
- **Flyway**나 **Liquibase**와 같은 전용 마이그레이션 도구를 사용하십시오.

## 테스트

- `@DataJpaTest`와 **Testcontainers**를 함께 사용하여 실제 운영 환경과 유사한 DB 환경에서 테스트하십시오.
- 로그(`org.hibernate.SQL=DEBUG`)를 통해 실행되는 SQL의 효율성을 주기적으로 점검하십시오.

**기억하십시오**: 엔티티는 가볍게, 쿼리는 의도적으로, 트랜잭션은 짧게 유지하십시오. N+1 문제는 페치 전략과 프로젝션으로 해결하고, 조회/쓰기 경로에 맞춰 인덱싱하십시오.
    
