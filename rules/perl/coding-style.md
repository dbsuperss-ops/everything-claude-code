---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Perl 코딩 스타일 (Coding Style)

> 이 문서는 [common/coding-style.md](../common/coding-style.md)의 규칙을 기반으로 Perl에 특화된 내용을 확장합니다.

## 표준 (Standards)

- 항상 `use v5.36` 라이브러리를 사용하십시오 (`strict`, `warnings`, `say`, 서브루틴 서명 기능이 활성화됩니다).
- 서브루틴 서명(Subroutine signatures) 기능을 사용하십시오 — `@_`를 수동으로 언팩하지 마십시오.
- 명시적인 개행 문자가 포함된 `print`보다 `say` 사용을 선호하십시오.

## 불변성 (Immutability)

- 모든 속성에 대해 `is => 'ro'`가 설정된 **Moo**와 `Types::Standard`를 사용하십시오.
- Blessed hashref를 직접 사용하지 말고, 항상 Moo/Moose 접근자(Accessors)를 사용하십시오.
- **OO 오버라이드 참고**: 계산된 읽기 전용 값을 위해 `builder` 또는 `default`가 포함된 Moo의 `has` 속성을 사용하는 것은 허용됩니다.

## 포매팅 (Formatting)

다음 설정을 포함한 **perltidy**를 사용하십시오:

```
-i=4    # 4칸 들여쓰기
-l=100  # 줄 길이 100자 제한
-ce     # Cuddled else (닫는 중괄호와 else를 같은 줄에 배치)
-bar    # 여는 중괄호는 항상 오른쪽에 배치
```

## 린팅 (Linting)

`core`, `pbp`, `security` 테마와 함께 중요도(Severity) 3 이상의 **perlcritic**을 사용하십시오.

```bash
perlcritic --severity 3 --theme 'core || pbp || security' lib/
```

## 참고 자료

현대적인 Perl 관례와 베스트 프랙티스에 대해서는 `perl-patterns` 스킬을 참조하십시오.
