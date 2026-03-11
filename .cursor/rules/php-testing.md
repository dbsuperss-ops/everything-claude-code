---
description: "공통 규칙을 확장하는 PHP 테스트"
globs: ["**/*.php", "**/phpunit.xml", "**/phpunit.xml.dist", "**/composer.json"]
alwaysApply: false
---
# PHP 테스트 (Testing)

> 이 문서는 공통 테스트 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## 프레임워크

기본 테스트 프레임워크로 **PHPUnit**을 사용하십시오. 프로젝트에서 이미 사용 중인 경우 **Pest**도 허용됩니다.

## 커버리지 (Coverage)

```bash
vendor/bin/phpunit --coverage-text
# 또는
vendor/bin/pest --coverage
```

## 테스트 구성

- 속도가 빠른 단위 테스트와 프레임워크/데이터베이스 통합 테스트를 분리하십시오.
- 수동으로 작성된 거대한 배열 대신 팩토리/빌더를 사용하여 픽스처(Fixtures)를 만드십시오.
- HTTP/컨트롤러 테스트는 전송 및 검증에만 집중하고, 비즈니스 규칙은 서비스 레벨 테스트로 이동하십시오.
