---
paths:
  - "**/*.php"
  - "**/phpunit.xml"
  - "**/phpunit.xml.dist"
  - "**/composer.json"
---
# PHP 테스트 (Testing)

> 이 문서는 [common/testing.md](../common/testing.md)의 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## 프레임워크

기본 테스트 프레임워크로 **PHPUnit**을 사용하십시오. 프로젝트에서 이미 사용 중인 경우 **Pest** 역시 허용됩니다.

## 커버리지

```bash
vendor/bin/phpunit --coverage-text
# 또는
vendor/bin/pest --coverage
```

CI(지속적 통합) 환경에서는 **pcov** 또는 **Xdebug** 사용을 선호하며, 커버리지 임계값(Thresholds)을 구두로 관리하기보다 CI 설정 파일에 명시적으로 기록하여 관리하십시오.

## 테스트 구성

- 빠른 속도의 단위 테스트와 프레임워크/데이터베이스 연동 테스트(Integration tests)를 분리하십시오.
- 픽스처(Fixtures)를 위해 거대한 수동 배열을 만드는 대신 팩토리(Factory)/빌더(Builders)를 사용하십시오.
- HTTP/컨트롤러 테스트는 전송 및 검증 로직에 집중하고, 비즈니스 규칙은 서비스 레벨 테스트에서 다루십시오.

## 참고 자료

리포지토리 전반의 RED -> GREEN -> REFACTOR 루프에 대해서는 `tdd-workflow` 스킬을 참조하십시오.
