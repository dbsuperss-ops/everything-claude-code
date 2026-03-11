---
name: perl-testing
description: Test2::V0, Test::More, prove 실행기, 모킹(Mocking), Devel::Cover를 이용한 커버리지 및 TDD 방법론을 포함하는 Perl 테스트 패턴 가이드입니다.
origin: ECC
---

# Perl 테스트 패턴 (Testing Patterns)

Test2::V0, Test::More, prove 및 TDD 방법론을 사용한 Perl 애플리케이션용 포괄적 테스트 전략입니다.

## 활성화 시점

- 새로운 Perl 코드를 작성할 때 (RED-GREEN-REFACTOR 패턴 준수)
- Perl 모듈이나 애플리케이션의 테스트 스위트를 설계할 때
- Perl 테스트 커버리지를 검토할 때
- Perl 테스트 인프라를 구축할 때
- 테스트를 Test::More에서 Test2::V0로 마이그레이션할 때
- 실패하는 Perl 테스트를 디버깅할 때

## TDD 워크플로우

항상 RED-GREEN-REFACTOR 주기를 따르십시오.

1. **RED**: 먼저 실패하는 테스트를 작성합니다 (`t/unit/` 디렉토리).
2. **GREEN**: 테스트를 통과시키기 위한 최소한의 구현 코드를 작성합니다 (`lib/` 디렉토리).
3. **REFACTOR**: 테스트 통과 상태를 유지하며 코드를 개선합니다.

실행: `prove -lv t/unit/calculator.t`

## 테스트 프레임워크

### 1. Test::More (기본 표준)

가장 널리 쓰이며 Perl 코어에 포함되어 있습니다.
- `is($got, $expected, '설명')`: 값 비교
- `ok($boolean, '설명')`: 참/거짓 확인
- `is_deeply($got, $expected, '설명')`: 복합 구조체 비교
- `like($got, qr/pattern/, '설명')`: 정규식 매칭

### 2. Test2::V0 (현대적 프레임워크)

Test::More를 대체하는 현대적 프레임워크로, 더 강력한 비교 도구와 명확한 에러 메시지를 제공합니다.
- `hash { field name => 'Alice'; etc(); }`: 해시의 일부 필드만 비교
- `array { item 'first'; etc(); }`: 배열 내용 비교
- `dies { code }`: 예외 발생 여부 테스트
- `subtest '설명' => sub { ... }`: 관련 테스트 그룹화

## 테스트 실행 (prove)

- `prove -l t/`: 모든 테스트 실행 (`-l`은 lib/를 포함)
- `prove -lv t/unit/user.t`: 특정 파일 상세 실행
- `prove -lr -j8 t/`: 8개 작업 병렬로 재귀적 실행
- `prove --state=failed`: 직전 실행에서 실패한 테스트만 실행

## 모킹 (Mocking)

`Test::MockModule`을 사용하여 외부 의존성(API, DB 등)을 격리하십시오.

```perl
subtest 'mock external API' => sub {
    my $mock = Test::MockModule->new('MyApp::API');
    $mock->mock(fetch_user => sub ($self, $id) {
        return { name => 'Mock User' };
    });
    # 테스트 코드...
    # $mock이 범위를 벗어나면 원본 메서드가 자동으로 복구됨
};
```

## 커버리지 (Devel::Cover)

코드의 어느 부분이 테스트되었는지 확인합니다.
- `cover -test`: 테스트 실행 및 커버리지 요약
- `cover -report html`: 상세한 브라우저 보고서 생성 (`cover_db/coverage.html`)

## 최선 관행 (Best Practices)

- **동작을 테스트하십시오**: 내부 구현이 아닌 입력에 따른 출력을 확인하십시오.
- **테스트 파일 이름을 명확하게 지으십시오**: `unit/`, `integration/` 등으로 구분하십시오.
- **독립성을 유지하십시오**: 서브 테스트 간에 상태를 공유하지 마십시오.
- **엣지 케이스를 포함하십시오**: 빈 문자열, `undef`, 제로 값 등을 테스트하십시오.
- **항상 `done_testing`으로 마무리하십시오**: 모든 테스트가 계획대로 실행되었음을 보장합니다.

**기억하십시오**: 테스트는 당신의 안전망입니다. 빠르고, 집중적이며, 독립적으로 유지하십시오. 새로운 프로젝트에는 Test2::V0와 prove를 사용하십시오.
