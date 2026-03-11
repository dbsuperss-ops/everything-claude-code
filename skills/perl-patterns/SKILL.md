---
name: perl-patterns
description: 견고하고 유지보수가 쉬운 Perl 애플리케이션 구축을 위한 Modern Perl 5.36+ 관용구, 최선 관행 및 규칙 가이드입니다.
origin: ECC
---

# Modern Perl 개발 패턴

견고하고 유지보수가 쉬운 애플리케이션 구축을 위한 관용적인(Idiomatic) Perl 5.36+ 패턴과 최선 관행을 안내합니다.

## 활성화 시점

- 새로운 Perl 코드나 모듈을 작성할 때
- Perl 코드를 검토하여 관용구 준수 여부를 확인할 때
- 기존의 레거시 Perl 코드를 현대적 표준으로 리팩토링할 때
- Perl 모듈 아키텍처를 설계할 때
- 5.36 이전 버전의 코드를 Modern Perl로 마이그레이션할 때

## 핵심 원칙

### 1. `v5.36` 프라그마(Pragma) 사용

단일 `use v5.36` 구문이 예전의 복잡한 초기 선언(Boilerplate)을 대체하며 `strict`, `warnings`, `signatures` 기능을 활성화합니다.

```perl
# 좋음: 현대적인 서두
use v5.36;

sub greet($name) {
    say "Hello, $name!";
}

# 나쁨: 레거시 방식
use strict;
use warnings;
use feature 'say', 'signatures';
no warnings 'experimental::signatures';

sub greet {
    my ($name) = @_;
    say "Hello, $name!";
}
```

### 2. 서브루틴 시그니처 (Subroutine Signatures)

명확성과 인자 개수 자동 확인을 위해 시그니처를 사용하십시오.

```perl
use v5.36;

# 좋음: 기본값이 포함된 시그니처
sub connect_db($host, $port = 5432, $timeout = 30) {
    # $host는 필수이며, 나머지는 기본값이 있음
}
```

### 3. 포스트픽스 역참조 (Postfix Dereferencing)

중첩된 구조체에서 가독성을 높이기 위해 포스트픽스 역참조 구문을 사용하십시오.

```perl
use v5.36;
my $data = { users => [...] };

# 좋음: 포스트픽스 역참조
my @users = $data->{users}->@*;
my %first = $data->{users}[0]->%*;

# 나쁨: 기존의 감싸는 방식 (복잡한 체인에서 읽기 힘듦)
my @users = @{ $data->{users} };
```

## 에러 처리

`eval/die` 패턴보다는 `Try::Tiny` 모듈을 사용하거나, 최신 버전(5.40+)에서는 네이티브 `try/catch`를 사용하십시오.

```perl
use v5.36;
use Try::Tiny;

my $user = try {
    $db->find($id) // die "User not found\n";
}
catch {
    warn "Failed: $_";
    undef;
};
```

## Moo를 이용한 현대적 객체 지향 (OO)

가볍고 현대적인 OO를 위해 Moo를 사용하십시오. 타입 검사(Types::Standard)와 함께 사용하면 더욱 견고해집니다.

```perl
package User;
use Moo;
use Types::Standard qw(Str Int);
use namespace::autoclean;

has name  => (is => 'ro', isa => Str, required => 1);
has age   => (is => 'ro', isa => Int, default => sub { 0 });

sub greet($self) {
    return "Hello, I'm " . $self->name;
}

1;
```

## 정규 표현식

가독성을 위해 `/x` 플래그와 이름이 지정된 캡처(Named Captures)를 사용하십시오.

```perl
my $log_re = qr{
    ^ (?<timestamp> \d{4}-\d{2}-\d{2} \s \d{2}:\d{2}:\d{2} )
    \s+ \[ (?<level> \w+ ) \]
    \s+ (?<message> .+ ) $
}x;

if ($line =~ $log_re) {
    say "Level: $+{level}, Message: $+{message}";
}
```

## 파일 I/O

보안과 명확성을 위해 항상 3-인자 `open`을 사용하고, `autodie`를 활용하여 에러 처리를 간소화하십시오. 복잡한 파일 작업에는 `Path::Tiny` 사용을 권장합니다.

```perl
use v5.36;
use autodie;

# 좋음
open my $fh, '<:encoding(UTF-8)', $path;

# Path::Tiny 활용
use Path::Tiny;
my $content = path($path)->slurp_utf8;
```

## 안티 패턴 주의

- **2-인자 `open`**: 셸 인젝션 리스크가 있습니다. 절대 사용하지 마십시오.
- **간접 객체 구문**: `new Foo(...)` 대신 `Foo->new(...)`를 사용하십시오.
- **모듈 로딩 시 문자열 `eval`**: 코드 인젝션 위험이 있습니다. `Module::Runtime`을 사용하십시오.
- **전역 변수에 대한 과도한 의존**: `our $VAR` 대신 상성이나 Moo 속성을 사용하십시오.

**기억하십시오**: Modern Perl은 깔끔하고, 읽기 쉬우며, 안전합니다. `use v5.36`을 서두에 배치하고, 객체 지향에는 Moo를, 복잡한 파일 작업에는 Path::Tiny를 활용하십시오.
