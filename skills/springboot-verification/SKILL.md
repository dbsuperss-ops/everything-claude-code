---
name: springboot-verification
description: Spring Boot 프로젝트를 위한 검증 루프 가이드입니다. 배포 또는 PR 전에 빌드, 정적 분석, 커버리지 확인, 보안 스캔 및 변경 사항 검토를 수행합니다.
origin: ECC
---

# Spring Boot 검증 루프 (Spring Boot Verification Loop)

Pull Request(PR) 전, 주요 변경 후, 그리고 배포 직전에 이 루프를 실행하십시오.

## 활성화 시점

- Spring Boot 서비스에 대한 PR을 올리기 전
- 대규모 리팩토링이나 의존성 업그레이드 후
- 스테이징 또는 운영 환경 배포 전 검증 시
- 전체 빌드 → 린트 → 테스트 → 보안 스캔 파이프라인을 실행할 때
- 테스트 커버리지가 임계치(예: 80% 이상)를 만족하는지 확인할 때

## 단계별 검증 절차

### 1단계: 빌드 (Build)
- `mvn clean verify -DskipTests` 또는 `./gradlew clean assemble -x test`를 실행하십시오.
- 빌드가 실패하면 즉시 중단하고 수정하십시오.

### 2단계: 정적 분석 (Static Analysis)
- **Maven**: `spotbugs`, `pmd`, `checkstyle` 플러그인을 사용하여 잠재적인 버그와 코드 스타일을 점검하십시오.
- **Gradle**: `checkstyleMain`, `pmdMain`, `spotbugsMain` 태스크를 활용하십시오.

### 3단계: 테스트 및 커버리지
- 단위 테스트(Mocking 활용)와 통합 테스트(Testcontainers 활용)를 모두 실행하십시오.
- `jacoco:report`를 통해 라인/브랜치 커버리지가 80% 이상인지 확인하십시오.
- 결과 보고서에는 총 테스트 수, 패스/실패 수, 커버리지 비율이 포함되어야 합니다.

### 4단계: 보안 스캔 (Security Scan)
- **의존성**: `dependency-check-maven` 등을 사용하여 알려진 취약점(CVE)이 있는지 확인하십시오.
- **시크릿**: 소스 코드나 설정 파일(`.yml`, `.properties`)에 비밀번호나 API 키가 하드코딩되어 있는지 `grep` 등으로 검색하십시오.
- **안티 패턴**: `System.out.println` 사용 여부나 예외 메시지 노출 여부, 와일드카드 CORS 설정 등을 점검하십시오.

### 5단계: 포맷팅 및 디프(Diff) 검토
- `spotless:apply`를 사용하여 코드를 자동 정렬하십시오.
- `git diff`를 통해 변경된 파일을 최종 검토하십시오. 디버깅 로그가 남아있는지, 트랜잭션과 검증 로직이 누락되지 않았는지 확인하십시오.

## 검증 결과 보고 양식 (예시)

- **빌드**: [성공/실패]
- **정적 분석**: [성공/실패] (SpotBugs/PMD/Checkstyle)
- **테스트**: [성공/실패] (성공X/전체Y, 커버리지 Z%)
- **보안**: [성공/실패] (취약점 발견 수: N)
- **종합 판정**: [준비 완료 / 수정 필요]

**기억하십시오**: 빠른 피드백이 나중의 큰 문제를 방지합니다. 검증 관문을 엄격하게 유지하고, 경고(Warning)를 가볍게 여기지 마십시오.
    
