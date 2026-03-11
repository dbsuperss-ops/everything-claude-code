---
paths:
  - "**/*.php"
  - "**/composer.json"
  - "**/phpstan.neon"
  - "**/phpstan.neon.dist"
  - "**/psalm.xml"
---
# PHP 후크 (Hooks)

> 이 문서는 [common/hooks.md](../common/hooks.md)의 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json`에서 다음을 구성하십시오:

- **Pint / PHP-CS-Fixer**: 편집된 `.php` 파일에 대해 자동 포매팅을 수행합니다.
- **PHPStan / Psalm**: 타입 정의가 포함된 코드베이스에서 PHP 편집 후 정적 분석을 실행합니다.
- **PHPUnit / Pest**: 동작에 영향을 주는 편집이 발생한 경우, 연관된 파일이나 모듈에 대해 타겟 테스트를 실행합니다.

## 경고 사항

- 편집된 파일에 `var_dump`, `dd`, `dump`, 또는 `die()`가 남아 있는 경우 경고를 표시합니다.
- PHP 파일 편집 시 Raw SQL이 추가되거나 CSRF/세션 보호 기능이 비활성화되는 경우 경고를 표시합니다.
