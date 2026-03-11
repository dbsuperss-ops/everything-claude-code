---
description: "공통 규칙을 확장하는 PHP 패턴"
globs: ["**/*.php", "**/composer.json"]
alwaysApply: false
---
# PHP 패턴 (Patterns)

> 이 문서는 공통 패턴 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## 가벼운 컨트롤러(Thin Controllers), 명시적 서비스

- 컨트롤러는 인증, 검증, 직렬화, 상태 코드와 같은 전송 계층의 역할에만 집중하도록 유지하십시오.
- 비즈니스 규칙은 HTTP 부트스트래핑 없이도 테스트하기 쉬운 애플리케이션/도메인 서비스로 이동시키십시오.

## DTO 및 값 객체 (Value Objects)

- 복잡한 구조의 연관 배열을 요청, 명령(Command) 및 외부 API 페이로드를 위한 DTO로 대체하십시오.
- 화폐, 식별자 및 제약 사항이 있는 개념에 대해 값 객체를 사용하십시오.

## 의존성 주입 (Dependency Injection)

- 프레임워크 전역 객체가 아닌 인터페이스나 좁은 범위의 서비스 규약(Contract)에 의존하십시오.
- 서비스 로케이터(Service-locator) 조회 없이도 서비스를 테스트할 수 있도록 생성자를 통해 협력 객체를 전달하십시오.
