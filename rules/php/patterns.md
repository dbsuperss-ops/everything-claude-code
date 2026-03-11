---
paths:
  - "**/*.php"
  - "**/composer.json"
---
# PHP 패턴 (Patterns)

> 이 문서는 [common/patterns.md](../common/patterns.md)의 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## 씬 컨트롤러 (Thin Controllers), 명시적 서비스

- 컨트롤러는 인증, 검증, 직렬화, 상태 코드 등 전송 계층의 역할에만 집중하도록 유지하십시오.
- 비즈니스 규칙은 HTTP 부트스트랩 없이도 테스트하기 쉬운 애플리케이션/도메인 서비스로 이동시키십시오.

## DTO 및 값 객체 (Value Objects)

- 요청, 명령어(Commands), 외부 API 페이로드 등을 위해 형태가 복잡한 연관 배열(Associative arrays) 대신 DTO를 사용하십시오.
- 금액, 식별자, 날짜 범위 등 제약 조건이 있는 개념들을 표현할 때는 값 객체(Value objects)를 사용하십시오.

## 의존성 주입 (Dependency Injection)

- 프레임워크 전역 변수(Globals)가 아닌, 인터페이스나 좁은 범위의 서비스 규약(Contracts)에 의존하도록 설계하십시오.
- 서비스 로케이터(Service-locator) 조회 없이도 서비스를 테스트할 수 있도록 의존성 요소를 생성자를 통해 전달하십시오.

## 경계 (Boundaries)

- 모델 레이어가 단순히 영속성 이상의 역할을 수행하는 경우, ORM 모델을 도메인 결정 로직으로부터 분리하십시오.
- 서드파티 SDK를 작은 어댑터(Adapters)로 래핑하여, 나머지 코드베이스가 외부 라이브러리가 아닌 여러분이 정의한 규약에 의존하도록 만드십시오.

## 참고 자료

엔드포인트 컨벤션 및 응답 형태 가이드에 대해서는 `api-design` 스킬을 참조하십시오.
