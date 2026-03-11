---
paths:
  - "**/*.php"
  - "**/composer.json"
---
# PHP 코딩 스타일 (Coding Style)

> 이 문서는 [common/coding-style.md](../common/coding-style.md)의 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## 표준 (Standards)

- **PSR-12** 포매팅 및 명명 컨벤션을 따릅니다.
- 애플리케이션 코드에서는 `declare(strict_types=1);` 사용을 선호하십시오.
- 새로운 코드 작성이 허용되는 모든 곳에서 스칼라 타입 힌트(Type hints), 리턴 타입 및 타입이 지정된 속성을 사용하십시오.

## 불변성 (Immutability)

- 서비스 경계를 넘나드는 데이터에 대해 불변 DTO 및 값 객체(Value objects)를 선호하십시오.
- 요청/응답 페이로드에 대해 가능한 경우 `readonly` 속성이나 불변 생성자를 사용하십시오.
- 단순한 맵 구조에는 배열을 유지하되, 비즈니스상 중요한 구조는 명시적인 클래스로 승격시키십시오.

## 포매팅 (Formatting)

- 포매팅을 위해 **PHP-CS-Fixer** 또는 **Laravel Pint**를 사용하십시오.
- 정적 분석을 위해 **PHPStan** 또는 **Psalm**을 사용하십시오.
- 로컬과 CI(지속적 통합) 환경에서 동일한 명령어를 실행할 수 있도록 Composer 스크립트를 관리하십시오.

## 에러 처리

- 비정상적인 상태에 대해서는 예외(Exception)를 던지십시오. 새로운 코드에서 `false`나 `null`을 숨겨진 에러 채널로 반환하는 것을 지양하십시오.
- 도메인 로직에 도달하기 전에 프레임워크/요청 입력을 검증된 DTO로 변환하십시오.

## 참고 자료

더 넓은 범위의 서비스/저장소 레이어링 가이드에 대해서는 `backend-patterns` 스킬을 참조하십시오.
