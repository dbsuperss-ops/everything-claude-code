---
name: springboot-verification
description: "Spring Boot 프로젝트 검증 사이클 가이드입니다. 빌드, 정적 분석, 테스트 커버리지 확인, 보안 스캔, PR 전 변경 사항 검토 과정을 다룹니다."
origin: ECC
---

# Spring Boot 검증 사이클

PR(Pull Request) 생성 전, 주요 변경 사항 적용 후, 그리고 배포 직전에 이 사이클을 실행하십시오.

## 적용 시점

* Spring Boot 서비스에서 PR을 올리기 전
* 대규모 리팩토링이나 의존성 라이브러리 업데이트 후
* 스테이징 또는 운영 환경 배포 전 최종 검증 시
* 빌드 → 린팅 → 테스트 → 보안 스캔 파이프라인을 전체 실행할 때
* 테스트 커버리지 목표치(80%+) 달성 여부를 확인할 때

## 단계별 검증 절차

### 1단계: 빌드 (Build)
```bash
# Maven
mvn clean verify -DskipTests
# Gradle
./gradlew clean assemble -x test
```
빌드에 실패하면 즉시 중단하고 수정하십시오.

### 2단계: 정적 분석 (Static Analysis)
SpotBugs, PMD, Checkstyle 등을 활용하여 코드 품질을 체크합니다.
```bash
mvn spotbugs:check pmd:check checkstyle:check
```

### 3단계: 테스트 및 커버리지 (Tests & Coverage)
코드의 논리적 결함을 찾고 테스트가 충분한지 확인합니다.
```bash
mvn test jacoco:report
# 또는
./gradlew test jacocoTestReport
```
* **단위 테스트**: Mockito를 사용하여 개별 서비스 로직을 격리하여 테스트합니다.
* **통합 테스트**: Testcontainers를 사용하여 H2가 아닌 실제 DB(Postgres 등) 환경에서 테스트하십시오.
* **API 테스트**: MockMvc를 사용하여 컨트롤러 계층과 HTTP 응답을 검증합니다.

### 4단계: 보안 스캔 (Security Scan)
* **의존성 취약점**: OWASP Dependency Check 등을 실행하여 알려진 CVE가 있는지 확인합니다.
* **비밀 정보 노출**: 소스 코드나 설정 파일(`.yml`, `.properties`)에 비밀번호나 API 키가 하드코딩되어 있는지 `grep` 등으로 스캔하십시오.
* **안티 패턴 체크**: `System.out.println` 사용 여부, 에러 응답에 원본 예외 메시지 노출 여부 등을 확인합니다.

### 5단계: 변경 사항 검토 (Diff Review)
```bash
git diff --stat
```
* 불필요한 디버그 로그가 남아있지 않은가?
* 적절한 HTTP 상태 코드와 의미 있는 에러 메시지를 반환하는가?
* 필요한 곳에 트랜잭션(`@Transactional`)과 검증(`@Valid`)이 적용되었는가?

---

## 검증 보고서 템플릿 (Verification Report)

```
[검증 보고서]
===================
빌드 결과:     [PASS/FAIL]
정적 분석:     [PASS/FAIL] (SpotBugs/Checkstyle 등)
테스트 결과:   [PASS/FAIL] (성공/전체, 커버리지 %)
보안 스캔:     [PASS/FAIL] (발견된 CVE 수: N)
변경 사항:     [수정된 파일 수]

최종 판정:     [READY / NOT READY]

수정 필요 사항:
1. ...
2. ...
```

**핵심**: 예기치 못한 당황스러운 상황보다 빠른 피드백이 낫습니다. 경고(Warning)를 실제 결함처럼 엄격하게 취급하여 프로덕션 품질을 유지하십시오.
