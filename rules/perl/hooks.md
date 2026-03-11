---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Perl 후크 (Hooks)

> 이 문서는 [common/hooks.md](../common/hooks.md)의 규칙을 기반으로 Perl에 특화된 내용을 확장합니다.

## PostToolUse 후크

`~/.claude/settings.json`에서 다음을 구성하십시오:

- **perltidy**: `.pl` 및 `.pm` 파일 편집 후 자동 포매팅 수행
- **perlcritic**: `.pm` 파일 편집 후 린트 체크 실행

## 경고 사항

- 스크립트가 아닌 `.pm` 파일에서 `print` 사용 시 경고 — `say` 또는 `Log::Any`와 같은 로깅 모듈을 사용하십시오.
