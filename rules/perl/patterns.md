---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Perl 패턴 (Patterns)

> 이 문서는 [common/patterns.md](../common/patterns.md)의 규칙을 기반으로 Perl에 특화된 내용을 확장합니다.

## 저장소(Repository) 패턴

인터페이스 뒤에 **DBI** 또는 **DBIx::Class**를 사용하십시오:

```perl
package MyApp::Repo::User;
use Moo;

has dbh => (is => 'ro', required => 1);

sub find_by_id ($self, $id) {
    my $sth = $self->dbh->prepare('SELECT * FROM users WHERE id = ?');
    $sth->execute($id);
    return $sth->fetchrow_hashref;
}
```

## DTO / 값 객체 (Value Objects)

**Types::Standard**를 포함한 **Moo** 클래스를 사용하십시오 (Python의 dataclass와 유사):

```perl
package MyApp::DTO::User;
use Moo;
use Types::Standard qw(Str Int);

has name  => (is => 'ro', isa => Str, required => 1);
has email => (is => 'ro', isa => Str, required => 1);
has age   => (is => 'ro', isa => Int);
```

## 리소스 관리

- 항상 `autodie`를 포함한 **3인자 open (three-arg open)** 형식을 사용하십시오.
- 파일 작업에는 **Path::Tiny**를 사용하십시오.

```perl
use autodie;
use Path::Tiny;

my $content = path('config.json')->slurp_utf8;
```

## 모듈 인터페이스

`Exporter 'import'`와 함께 `@EXPORT_OK`를 사용하십시오. — `@EXPORT`는 절대로 사용하지 마십시오:

```perl
use Exporter 'import';
our @EXPORT_OK = qw(parse_config validate_input);
```

## 의존성 관리

재현 가능한 설치를 위해 **cpanfile** + **carton**을 사용하십시오:

```bash
carton install
carton exec prove -lr t/
```

## 참고 자료

현대적인 Perl 패턴과 관례에 대해서는 `perl-patterns` 스킬을 참조하십시오.
