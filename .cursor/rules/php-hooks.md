---
description: "공통 규칙을 확장하는 PHP 후크(Hooks)"
globs: ["**/*.php", "**/composer.json", "**/phpstan.neon", "**/phpstan.neon.dist", "**/psalm.xml"]
alwaysApply: false
---
# PHP 후크 (Hooks)

> 이 문서는 공통 후크 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json` 파일에서 다음 항목을 구성하십시오:

- **Pint / PHP-CS-Fixer**: 편집된 `.php` 파일을 자동으로 포매팅합니다.
- **PHPStan / Psalm**: 타입이 지정된 코드베이스에서 PHP 편집 후 정적 분석을 실행합니다.
- **PHPUnit / Pest**: 편집 내용이 동작에 영향을 미치는 경우, 수정된 파일이나 모듈에 대해 타겟 테스트를 실행합니다.

## 경고 사항 (Warnings)

- 편집된 파일에 `var_dump`, `dd`, `dump`, 또는 `die()`가 남아 있는 경우 경고합니다.
- 편집된 PHP 파일에서 원시 SQL(Raw SQL)을 추가하거나 CSRF/세션 보호 기능을 비활성화하는 경우 경고합니다.
