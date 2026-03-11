---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Perl 테스트 (Testing)

> 이 문서는 [common/testing.md](../common/testing.md)의 규칙을 기반으로 Perl에 특화된 내용을 확장합니다.

## 프레임워크

신규 프로젝트에는 `Test::More` 대신 **Test2::V0**를 사용하십시오:

```perl
use Test2::V0;

is($result, 42, '정답이 올바름');

done_testing;
```

## 실행 도구 (Runner)

```bash
prove -l t/              # lib/ 디렉토리를 @INC에 추가
prove -lr -j8 t/         # 재귀적 실행, 8개 작업을 병렬로 수행
```

`lib/`가 `@INC`에 포함되도록 항상 `-l` 옵션을 사용하십시오.

## 커버리지

**Devel::Cover**를 사용하며, 80% 이상의 커버리지를 목표로 하십시오.

```bash
cover -test
```

## 모의 객체 (Mocking)

- **Test::MockModule** — 기존 모듈의 메서드를 모킹할 때 사용
- **Test::MockObject** — 테스트용 가짜 객체(Test doubles)를 처음부터 생성할 때 사용

## 주의 사항

- 테스트 파일은 항상 `done_testing`으로 종료하십시오.
- `prove` 실행 시 `-l` 플래그를 절대 잊지 마십시오.

## 참고 자료

Test2::V0, prove, Devel::Cover를 활용한 상세한 Perl TDD 패턴에 대해서는 `perl-testing` 스킬을 참조하십시오.
