# 세션 명령어 (Sessions Command)

Claude Code 세션 이력을 관리합니다. `~/.claude/sessions/`에 저장된 세션의 목록 확인, 로드, 별칭(alias) 설정 및 편집을 수행합니다.

## 사용법

`/sessions [list|load|alias|info|help] [옵션]`

## 주요 동작

### 세션 목록 확인 (List Sessions)

메타데이터, 필터링, 페이지네이션과 함께 모든 세션을 표시합니다.

```bash
/sessions                              # 모든 세션 나열 (기본값)
/sessions list                         # 위와 동일
/sessions list --limit 10              # 10개의 세션만 표시
/sessions list --date 2026-02-01       # 날짜별 필터링
/sessions list --search abc            # 세션 ID로 검색
```

**스크립트:**
```bash
node -e "
const sm = require((process.env.CLAUDE_PLUGIN_ROOT||require('path').join(require('os').homedir(),'.claude'))+'/scripts/lib/session-manager');
const aa = require((process.env.CLAUDE_PLUGIN_ROOT||require('path').join(require('os').homedir(),'.claude'))+'/scripts/lib/session-aliases');

const result = sm.getAllSessions({ limit: 20 });
const aliases = aa.listAliases();
const aliasMap = {};
for (const a of aliases) aliasMap[a.sessionPath] = a.name;

console.log('세션 목록 (총 ' + result.total + '개 중 ' + result.sessions.length + '개 표시):');
console.log('');
console.log('ID        날짜        시간     크기     줄 수   별칭');
console.log('────────────────────────────────────────────────────');

for (const s of result.sessions) {
  const alias = aliasMap[s.filename] || '';
  const size = sm.getSessionSize(s.sessionPath);
  const stats = sm.getSessionStats(s.sessionPath);
  const id = s.shortId === 'no-id' ? '(없음)' : s.shortId.slice(0, 8);
  const time = s.modifiedTime.toTimeString().slice(0, 5);

  console.log(id.padEnd(8) + ' ' + s.date + '  ' + time + '   ' + size.padEnd(7) + '  ' + String(stats.lineCount).padEnd(5) + '  ' + alias);
}
"
```

### 세션 로드 (Load Session)

세션의 내용을 로드하여 표시합니다 (ID 또는 별칭 사용).

```bash
/sessions load <id|별칭>             # 세션 로드
/sessions load 2026-02-01             # 날짜로 로드 (ID가 없는 세션의 경우)
/sessions load a1b2c3d4               # 단축 ID로 로드
/sessions load my-alias               # 별칭으로 로드
```

### 별칭 생성 (Create Alias)

세션에 기억하기 쉬운 별칭을 부여합니다.

```bash
/sessions alias <id> <이름>           # 별칭 생성
/sessions alias 2026-02-01 today-work # "today-work"라는 이름의 별칭 생성
```

### 별칭 제거 (Remove Alias)

기존 별칭을 삭제합니다.

```bash
/sessions alias --remove <이름>        # 별칭 제거
/sessions unalias <이름>               # 위와 동일
```

### 세션 정보 (Session Info)

세션에 대한 자세한 정보를 표시합니다.

```bash
/sessions info <id|별칭>              # 세션 상세 정보 표시
```

### 별칭 목록 확인 (List Aliases)

모든 세션 별칭을 표시합니다.

```bash
/sessions aliases                      # 모든 별칭 나열
```

## 인자 (Arguments)

$인자:
- `list [옵션]` - 세션 목록 표시
  - `--limit <n>` - 표시할 최대 세션 수 (기본값: 50)
  - `--date <YYYY-MM-DD>` - 날짜별 필터링
  - `--search <패턴>` - 세션 ID 내 패턴 검색
- `load <id|별칭>` - 세션 내용 로드
- `alias <id> <이름>` - 세션 별칭 생성
- `alias --remove <이름>` - 별칭 제거
- `unalias <이름>` - 위와 동일
- `info <id|별칭>` - 세션 통계 정보 표시
- `aliases` - 모든 별칭 나열
- `help` - 도움말 표시

## 사용 예시

```bash
# 모든 세션 나열
/sessions list

# 오늘 세션에 대한 별칭 생성
/sessions alias 2026-02-01 today

# 별칭으로 세션 로드
/sessions load today

# 세션 정보 확인
/sessions info today

# 별칭 제거
/sessions alias --remove today

# 모든 별칭 확인
/sessions aliases
```

## 참고 사항

- 세션은 `~/.claude/sessions/` 경로에 마크다운 파일로 저장됩니다.
- 별칭 정보는 `~/.claude/session-aliases.json`에 저장됩니다.
- 세션 ID는 단축해서 사용할 수 있습니다 (보통 처음 4~8자 정도면 식별 가능합니다).
- 자주 참조하는 세션은 별칭을 사용하는 것이 편리합니다.
