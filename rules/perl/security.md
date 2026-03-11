---
paths:
  - "**/*.pl"
  - "**/*.pm"
  - "**/*.t"
  - "**/*.psgi"
  - "**/*.cgi"
---
# Perl 보안 (Security)

> 이 문서는 [common/security.md](../common/security.md)의 규칙을 기반으로 Perl에 특화된 내용을 확장합니다.

## 테인트 모드 (Taint Mode)

- 모든 CGI 및 웹 기반 스크립트에는 `-T` 플래그를 사용하십시오.
- 외부 명령어를 실행하기 전에 `%ENV` (`$ENV{PATH}`, `$ENV{CDPATH}` 등)를 반드시 정제하십시오.

## 입력값 검증

- 테인트 해제(Untainting)를 위해 허용 목록(Allowlist) 기반의 정규표현식을 사용하십시오. — `/(.*)/s`는 절대로 사용하지 마십시오.
- 모든 사용자 입력을 명시적인 패턴으로 검증하십시오:

```perl
if ($input =~ /\A([a-zA-Z0-9_-]+)\z/) {
    my $clean = $1;
}
```

## 파일 입출력 (File I/O)

- **오직 3인자 open (Three-arg open)**만 사용하십시오. — 2인자 open은 절대로 사용하지 마십시오.
- `Cwd::realpath`를 사용하여 경로 탐색(Path traversal) 취약점을 방지하십시오:

```perl
use Cwd 'realpath';
my $safe_path = realpath($user_path);
die "경로 탐색 위험 감지" unless $safe_path =~ m{\A/allowed/directory/};
```

## 프로세스 실행

- **리스트 형식의 `system()`**을 사용하십시오. — 단일 문자열 형식은 절대로 사용하지 마십시오.
- 출력을 캡처하려면 **IPC::Run3**를 사용하십시오.
- 변수 삽입(Interpolation)이 포함된 백틱(Backticks)을 사용하지 마십시오.

```perl
system('grep', '-r', $pattern, $directory);  # 안전함
```

## SQL 인젝션 방지

항상 DBI 플레이스홀더(Placeholders)를 사용하십시오. — SQL 문 내에 변수를 직접 삽입하지 마십시오:

```perl
my $sth = $dbh->prepare('SELECT * FROM users WHERE email = ?');
$sth->execute($email);
```

## 보안 스캐닝

중요도(Severity) 4 이상에서 `security` 테마와 함께 **perlcritic**을 실행하십시오.

```bash
perlcritic --severity 4 --theme security lib/
```

## 참고 자료

 Per의 종합적인 보안 패턴, 테인트 모드 및 안전한 I/O 관계에 대해서는 `perl-security` 스킬을 참조하십시오.
