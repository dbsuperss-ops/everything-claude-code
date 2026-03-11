---
name: perl-security
description: 테인트 모드, 입력값 검증, 안전한 프로세스 실행, DBI 매개변수화된 쿼리, 웹 보안(XSS/SQLi/CSRF) 및 perlcritic 보안 정책을 포함하는 종합적인 Perl 보안 가이드입니다.
origin: ECC
---

# Perl 보안 패턴 (Security Patterns)

입력값 검증, 인젝션 방지 및 안전한 코딩 관행을 포함하는 Perl 애플리케이션용 종합 보안 가이드라인입니다.

## 활성화 시점

- Perl 애플리케이션에서 사용자 입력을 처리할 때
- Perl 웹 애플리케이션(CGI, Mojolicious, Dancer2, Catalyst 등)을 구축할 때
- 보안 취약점에 대해 Perl 코드를 리뷰할 때
- 사용자로부터 제공받은 경로를 사용하여 파일 작업을 수행할 때
- Perl에서 시스템 명령어를 실행할 때
- DBI 데이터베이스 쿼리를 작성할 때

## 작동 방식

테인트(Taint)를 인식하는 입력 경계 설정부터 시작하여 외부로 확장해 나갑니다. 입력을 검증하고 테이트를 해제(Untaint)하며, 파일 시스템 및 프로세스 실행을 제한하고, 모든 곳에서 매개변수화된 DBI 쿼리를 사용하십시오. 아래 예시들은 사용자 입력, 셸 또는 네트워크와 접촉하는 Perl 코드를 배포하기 전에 이 스킬이 적용하기를 기대하는 안전한 기본 설정들을 보여줍니다.

## 테인트 모드 (Taint Mode)

Perl의 테인트 모드(`-T`)는 외부 소스로부터 온 데이터를 추적하고, 명시적인 검증 없이 위험한 작업에 사용되는 것을 방지합니다.

### 테인트 모드 활성화

```perl
#!/usr/bin/perl -T
use v5.36;

# 테인트된 데이터: 프로그램 외부에서 온 모든 것
my $input    = $ARGV[0];        # 테인트됨
my $env_path = $ENV{PATH};      # 테인트됨
my $form     = <STDIN>;         # 테인트됨
my $query    = $ENV{QUERY_STRING}; # 테인트됨

# PATH 조기 정제 (테인트 모드에서 필수)
$ENV{PATH} = '/usr/local/bin:/usr/bin:/bin';
delete @ENV{qw(IFS CDPATH ENV BASH_ENV)};
```

### 테인트 해제(Untainting) 패턴

```perl
use v5.36;

# 좋은 예: 특정 정규표현식으로 검증하고 테인트 해제
sub untaint_username($input) {
    if ($input =~ /^([a-zA-Z0-9_]{3,30})$/) {
        return $1;  # $1은 테인트가 해제된 상태임
    }
    die "유효하지 않은 사용자 이름: 3-30자의 영문 숫자로 구성되어야 합니다\n";
}

# 좋은 예: 파일 경로 검증 및 테인트 해제
sub untaint_filename($input) {
    if ($input =~ m{^([a-zA-Z0-9._-]+)$}) {
        return $1;
    }
    die "유효하지 않은 파일 이름: 안전하지 않은 문자가 포함되어 있습니다\n";
}

# 나쁜 예: 지나치게 허용적인 테인트 해제 (기능 무력화)
sub bad_untaint($input) {
    $input =~ /^(.*)$/s;
    return $1;  # 모든 것을 허용함 — 의미 없음
}
```

## 입력값 검증

### 차단 목록(Blocklist)보다 허용 목록(Allowlist)

```perl
use v5.36;

# 좋은 예: 허용 목록 — 허용되는 항목을 정확히 정의
sub validate_sort_field($field) {
    my %allowed = map { $_ => 1 } qw(name email created_at updated_at);
    die "유효하지 않은 정렬 필드: $field\n" unless $allowed{$field};
    return $field;
}

# 좋은 예: 특정 패턴으로 검증
sub validate_email($email) {
    if ($email =~ /^([a-zA-Z0-9._%+-]+\@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})$/) {
        return $1;
    }
    die "유효하지 않은 이메일 주소\n";
}

sub validate_integer($input) {
    if ($input =~ /^(-?\d{1,10})$/) {
        return $1 + 0;  # 숫자로 강제 변환
    }
    die "유효하지 않은 정수\n";
}

# 나쁜 예: 차단 목록 — 항상 불완전함
sub bad_validate($input) {
    die "Invalid" if $input =~ /[<>"';&|]/;  # 인코딩된 공격 시도를 놓칠 수 있음
    return $input;
}
```

### 길이 제한

```perl
use v5.36;

sub validate_comment($text) {
    die "댓글 내용은 필수입니다\n"        unless length($text) > 0;
    die "댓글이 10,000자를 초과합니다\n" if length($text) > 10_000;
    return $text;
}
```

## 안전한 정규표현식

### ReDoS 방지

겹치는 패턴에 중첩된 수량자(Quantifiers)를 사용하면 치명적인 백트래킹(Backtracking)이 발생할 수 있습니다.

```perl
use v5.36;

# 나쁜 예: ReDoS에 취약 (지수적 백트래킹 발생)
my $bad_re = qr/^(a+)+$/;           # 중첩된 수량자
my $bad_re2 = qr/^([a-zA-Z]+)*$/;   # 클래스에 대한 중첩된 수량자
my $bad_re3 = qr/^(.*?,){10,}$/;    # 탐욕적(Greedy)/게으른(Lazy) 수량자 조합 반복

# 좋은 예: 중첩 없이 재작성
my $good_re = qr/^a+$/;             # 단일 수량자
my $good_re2 = qr/^[a-zA-Z]+$/;     # 클래스에 대한 단일 수량자

# 좋은 예: 백트래킹 방지를 위해 소유격 수량자(Possessive quantifiers) 또는 원자적 그룹(Atomic groups) 사용
my $safe_re = qr/^[a-zA-Z]++$/;             # 소유격 수량자 (5.10 이상)
my $safe_re2 = qr/^(?>a+)$/;                # 원자적 그룹

# 좋은 예: 신뢰할 수 없는 패턴에 대해 타임아웃 강제 적용
use POSIX qw(alarm);
sub safe_match($string, $pattern, $timeout = 2) {
    my $matched;
    eval {
        local $SIG{ALRM} = sub { die "정규표현식 타임아웃\n" };
        alarm($timeout);
        $matched = $string =~ $pattern;
        alarm(0);
    };
    alarm(0);
    die $@ if $@;
    return $matched;
}
```

## 안전한 파일 작업

### 3인자 open (Three-Argument Open)

```perl
use v5.36;

# 좋은 예: 3인자 open, 어휘적 파일 핸들(Lexical filehandle), 반환값 확인
sub read_file($path) {
    open my $fh, '<:encoding(UTF-8)', $path
        or die "'$path' 파일을 열 수 없습니다: $!\n";
    local $/;
    my $content = <$fh>;
    close $fh;
    return $content;
}

# 나쁜 예: 사용자 데이터가 포함된 2인자 open (명령어 인젝션 유발)
sub bad_read($path) {
    open my $fh, $path;        # 만약 $path = "|rm -rf /"라면 명령어가 실행됨!
    open my $fh, "< $path";   # 셸 메타문자 인젝션 리스크
}
```

### TOCTOU 방지 및 경로 탐색 (Path Traversal)

```perl
use v5.36;
use Fcntl qw(:DEFAULT :flock);
use File::Spec;
use Cwd qw(realpath);

# 원자적 파일 생성
sub create_file_safe($path) {
    sysopen(my $fh, $path, O_WRONLY | O_CREAT | O_EXCL, 0600)
        or die "'$path' 파일을 생성할 수 없습니다: $!\n";
    return $fh;
}

# 경로가 허용된 디렉토리 내에 머무는지 검증
sub safe_path($base_dir, $user_path) {
    my $real = realpath(File::Spec->catfile($base_dir, $user_path))
        // die "경로가 존재하지 않습니다\n";
    my $base_real = realpath($base_dir)
        // die "기준 디렉토리가 존재하지 않습니다\n";
    die "경로 탐색 시도가 차단되었습니다\n" unless $real =~ /^\Q$base_real\E(?:\/|\z)/;
    return $real;
}
```

임시 파일에는 `File::Temp` (`tempfile(UNLINK => 1)`)를 사용하고, 경쟁 조건(Race conditions) 방지를 위해 `flock(LOCK_EX)`을 사용하십시오.

## 안전한 프로세스 실행

### 리스트 형식의 system 및 exec

```perl
use v5.36;

# 좋은 예: 리스트 형식 — 셸 삽입(Interpolation)이 발생하지 않음
sub run_command(@cmd) {
    system(@cmd) == 0
        or die "명령 실행 실패: @cmd\n";
}

run_command('grep', '-r', $user_pattern, '/var/log/app/');

# 좋은 예: IPC::Run3를 사용하여 안전하게 출력 캡처
use IPC::Run3;
sub capture_output(@cmd) {
    my ($stdout, $stderr);
    run3(\@cmd, \undef, \$stdout, \$stderr);
    if ($?) {
        die "명령 실행 실패 (종료 코드 $?): $stderr\n";
    }
    return $stdout;
}

# 나쁜 예: 문자열 형식 — 셸 인젝션 위험!
sub bad_search($pattern) {
    system("grep -r '$pattern' /var/log/app/");  # 만약 $pattern = "'; rm -rf / #"라면...
}

# 나쁜 예: 삽입 구문이 포함된 백틱 (Backticks)
my $output = `ls $user_dir`;   # 셸 인젝션 리스크
```

또한 외부 명령어로부터 stdout/stderr를 안전하게 캡처하기 위해 `Capture::Tiny`를 사용하십시오.

## SQL 인젝션 방지

### DBI 플레이스홀더 (Placeholders)

```perl
use v5.36;
use DBI;

my $dbh = DBI->connect($dsn, $user, $pass, {
    RaiseError => 1,
    PrintError => 0,
    AutoCommit => 1,
});

# 좋은 예: 매개변수화된 쿼리 — 항상 플레이스홀더를 사용하십시오
sub find_user($dbh, $email) {
    my $sth = $dbh->prepare('SELECT * FROM users WHERE email = ?');
    $sth->execute($email);
    return $sth->fetchrow_hashref;
}

sub search_users($dbh, $name, $status) {
    my $sth = $dbh->prepare(
        'SELECT * FROM users WHERE name LIKE ? AND status = ? ORDER BY name'
    );
    $sth->execute("%$name%", $status);
    return $sth->fetchall_arrayref({});
}

# 나쁜 예: SQL 문에 문자열 직접 삽입 (SQLi 취약점!)
sub bad_find($dbh, $email) {
    my $sth = $dbh->prepare("SELECT * FROM users WHERE email = '$email'");
    # 만약 $email = "' OR 1=1 --"이라면 모든 사용자가 반환됨
    $sth->execute;
    return $sth->fetchrow_hashref;
}
```

### 동적 컬럼 허용 목록 (Allowlists)

```perl
use v5.36;

# 좋은 예: 허용 목록을 기준으로 컬럼 이름 검증
sub order_by($dbh, $column, $direction) {
    my %allowed_cols = map { $_ => 1 } qw(name email created_at);
    my %allowed_dirs = map { $_ => 1 } qw(ASC DESC);

    die "유효하지 않은 컬럼: $column\n"    unless $allowed_cols{$column};
    die "유효하지 않은 정렬 방향: $direction\n" unless $allowed_dirs{uc $direction};

    my $sth = $dbh->prepare("SELECT * FROM users ORDER BY $column $direction");
    $sth->execute;
    return $sth->fetchall_arrayref({});
}

# 나쁜 예: 사용자가 선택한 컬럼을 직접 삽입
sub bad_order($dbh, $column) {
    $dbh->prepare("SELECT * FROM users ORDER BY $column");  # SQLi!
}
```

### DBIx::Class (ORM 보안)

```perl
use v5.36;

# DBIx::Class는 안전한 매개변수화된 쿼리를 생성합니다
my @users = $schema->resultset('User')->search({
    status => 'active',
    email  => { -like => '%@example.com' },
}, {
    order_by => { -asc => 'name' },
    rows     => 50,
});
```

## 웹 보안 (Web Security)

### XSS 방지

```perl
use v5.36;
use HTML::Entities qw(encode_entities);
use URI::Escape qw(uri_escape_utf8);

# 좋은 예: HTML 컨텍스트에 대해 출력 인코딩
sub safe_html($user_input) {
    return encode_entities($user_input);
}

# 좋은 예: URL 컨텍스트에 대해 인코딩
sub safe_url_param($value) {
    return uri_escape_utf8($value);
}

# 좋은 예: JSON 컨텍스트에 대해 인코딩
use JSON::MaybeXS qw(encode_json);
sub safe_json($data) {
    return encode_json($data);  # 이스케이프 처리를 수행함
}

# 템플릿 자동 이스케이프 (Mojolicious)
# <%= $user_input %>   — 자동 이스케이프됨 (안전)
# <%== $raw_html %>    — 생(Raw) 출력 (위험, 신뢰할 수 있는 콘텐츠에만 사용)

# 템플릿 자동 이스케이프 (Template Toolkit)
# [% user_input | html %]  — 명시적인 HTML 인코딩

# 나쁜 예: HTML 블록 내 생(Raw) 출력
sub bad_html($input) {
    print "<div>$input</div>";  # $input에 <script>가 포함된 경우 XSS 취약
}
```

### CSRF 보호

```perl
use v5.36;
use Crypt::URandom qw(urandom);
use MIME::Base64 qw(encode_base64url);

sub generate_csrf_token() {
    return encode_base64url(urandom(32));
}
```

토큰 검증 시 상수 시간 비교 (Constant-time comparison) 방식을 사용하십시오. 대부분의 웹 프레임워크(Mojolicious, Dancer2, Catalyst)는 내장된 CSRF 보호 기능을 제공하므로 가급적 이를 사용하십시오.

### 세션 및 헤더 보안

```perl
use v5.36;

# Mojolicious 세션 및 헤더 설정 예시
$app->secrets(['매우-긴-랜덤-비밀-값-주기적으로-교체 권장']);
$app->sessions->secure(1);          # HTTPS 전용
$app->sessions->samesite('Lax');

$app->hook(after_dispatch => sub ($c) {
    $c->res->headers->header('X-Content-Type-Options' => 'nosniff');
    $c->res->headers->header('X-Frame-Options'        => 'DENY');
    $c->res->headers->header('Content-Security-Policy' => "default-src 'self'");
    $c->res->headers->header('Strict-Transport-Security' => 'max-age=31536000; includeSubDomains');
});
```

## 출력 인코딩

출력 상황에 맞춰 항상 인코딩하십시오: HTML의 경우 `HTML::Entities::encode_entities()`, URL의 경우 `URI::Escape::uri_escape_utf8()`, JSON의 경우 `JSON::MaybeXS::encode_json()`을 사용하십시오.

## CPAN 모듈 보안

- cpanfile에서 **버전을 고정**하십시오: `requires 'DBI', '== 1.643';`
- **관리되고 있는 모듈을 선호**하십시오: MetaCPAN에서 최근 배포 내역을 확인하십시오.
- **의존성을 최소화**하십시오: 각각의 의존성은 공격 표면(Attack surface)이 될 수 있습니다.

## 보안 도구 (Security Tooling)

### perlcritic 보안 정책

```ini
# .perlcriticrc — 보안 중심 구성 예시
severity = 3
theme = security + core

# 3인자 open 강제
[InputOutput::RequireThreeArgOpen]
severity = 5

# 시스템 호출 결과 확인 강제
[InputOutput::RequireCheckedSyscalls]
functions = :builtins
severity = 4

# 문자열 eval 금지
[BuiltinFunctions::ProhibitStringyEval]
severity = 5

# 백틱(Backtick) 연산자 금지
[InputOutput::ProhibitBacktickOperators]
severity = 4

# CGI에서 테인트 체크 강제
[Modules::RequireTaintChecking]
severity = 5

# 2인자 open 금지
[InputOutput::ProhibitTwoArgOpen]
severity = 5

# 베어워드(Bare-word) 파일 핸들 금지
[InputOutput::ProhibitBarewordFileHandles]
severity = 5
```

### perlcritic 실행

```bash
# 파일 하나만 체크
perlcritic --severity 3 --theme security lib/MyApp/Handler.pm

# 전체 프로젝트 체크
perlcritic --severity 3 --theme security lib/

# CI 연동
perlcritic --severity 4 --theme security --quiet lib/ || exit 1
```

## 신속 보안 체크리스트

| 점검 항목 | 확인 사항 |
|---|---|
| 테인트 모드 | CGI/웹 스크립트에 `-T` 플래그 설정 여부 |
| 입력값 검증 | 허용 목록 패턴 및 길이 제한 적용 여부 |
| 파일 작업 | 3인자 open 및 경로 탐색 방지 체크 여부 |
| 프로세스 실행 | 리스트 형식의 system 사용 및 셸 삽입 부재 여부 |
| SQL 쿼리 | DBI 플레이스홀더 사용 및 직접 삽입 부재 여부 |
| HTML 출력 | `encode_entities()` 또는 템플릿 자동 이스케이프 적용 여부 |
| CSRF 토큰 | 생성 및 상태 변경 요청 시 검증 수행 여부 |
| 세션 설정 | Secure, HttpOnly, SameSite 쿠키 설정 여부 |
| HTTP 헤더 | CSP, X-Frame-Options, HSTS 적용 여부 |
| 의존성 관리 | 버전 고정 및 모듈 감사 수행 여부 |
| 정규표현식 안전성 | 중첩 수량자 부재 및 앵커 패턴 사용 여부 |
| 에러 메시지 | 사용자에게 스택 트레이스나 경로가 노출되지 않는지 여부 |

## 안티 패턴 (Anti-Patterns)

```perl
# 1. 사용자 데이터가 포함된 2인자 open (명령어 인젝션)
open my $fh, $user_input;               # 치명적인 취약점

# 2. 문자열 형식의 system (셸 인젝션)
system("convert $user_file output.png"); # 치명적인 취약점

# 3. SQL 문자열 삽입
$dbh->do("DELETE FROM users WHERE id = $id");  # SQLi

# 4. 사용자 입력이 포함된 eval (코드 인젝션)
eval $user_code;                         # 원격 코드 실행(RCE)

# 5. 정제 없이 $ENV 신뢰
my $path = $ENV{UPLOAD_DIR};             # 조작 가능성 있음
system("ls $path");                      # 이중 취약점

# 6. 검증 없이 테인트 해제
($input) = $input =~ /(.*)/s;           # 무분별한 테인트 해제 — 목적 상실

# 7. HTML 내 보정되지 않은 사용자 데이터 출력
print "<div>반갑습니다, $username!</div>";  # XSS

# 8. 검증되지 않은 리다이렉트 (Redirects)
print $cgi->redirect($user_url);         # 오픈 리다이렉트(Open redirect)
```

**기억하십시오**: Perl의 유연함은 강력하지만 그만큼 절제가 필요합니다. 웹 지향 코드에는 테인트 모드를 사용하고, 허용 목록을 통해 모든 입력을 검증하며, 모든 쿼리에 DBI 플레이스홀더를 사용하고, 상황에 맞게 모든 출력을 인코딩하십시오. 심층 방어(Defense in depth)를 항상 염두에 두십시오. 하나의 보안 계층에만 의존해서는 안 됩니다.
