# Claude Code 요약 가이드

![Header: Anthropic Hackathon Winner - Claude Code Tips & Tricks](../../assets/images/shortform/00-header.png)

***

**저는 지난 2월 실험 단계부터 Claude Code를 사용해 왔으며, [@DRodriguezFX](https://x.com/DRodriguezFX)와 함께 [zenith.chat](https://zenith.chat)을 만들어 Anthropic x Forum Ventures 해커톤에서 우승했습니다. 오직 Claude Code만을 사용해서 말이죠.**

10개월간 매일 사용하며 정립한 저의 전체 설정(스킬, 후크, 서브 에이전트, MCP, 플러그인 등)과 실제로 효과가 있었던 방법들을 공유합니다.

***

## 스킬(Skills) 및 명령어(Commands)

스킬은 특정 범위와 프로세스로 제한된 '규칙'과 같습니다. 특정 워크플로우를 실행해야 할 때 사용하는 프롬프트의 단축어라고 이해하시면 됩니다.

Opus 4.5로 긴 코딩 작업을 마친 후 쓰지 않는 코드나 흩어진 `.md` 파일들을 정리하고 싶으신가요? `/refactor-clean`을 실행하세요. 테스트가 필요한가요? `/tdd`, `/e2e`, `/test-coverage`가 있습니다. 스킬에는 '코드 맵'을 포함할 수도 있습니다. 이는 Claude가 컨텍스트를 소모하며 탐색할 필요 없이 코드베이스를 빠르게 훑어볼 수 있게 해주는 방식입니다.

![터미널에서 체이닝된 명령어 실행 모습](../../assets/images/shortform/02-chaining-commands.jpeg)
*명령어들을 체이닝하여 실행하기*

명령어는 슬래시(`/`) 명령을 통해 실행되는 스킬입니다. 스킬과 겹치는 부분이 있지만 저장 위치가 다릅니다:

* **스킬(Skills)**: `~/.claude/skills/` - 더 넓은 범위의 워크플로우 정의
* **명령어(Commands)**: `~/.claude/commands/` - 빠르게 실행 가능한 프롬프트

```bash
# 스킬 구조 예시
~/.claude/skills/
  pmx-guidelines.md      # 프로젝트별 패턴
  coding-standards.md    # 언어별 베스트 프랙티스
  tdd-workflow/          # README.md를 포함한 다중 파일 스킬
  security-review/       # 체크리스트 기반 스킬
```

***

## 후크(Hooks)

후크는 특정 이벤트가 발생할 때 실행되는 트리거 기반 자동화입니다. 스킬과 달리 도구 호출(Tool Call)이나 생명주기 이벤트에 결합됩니다.

**후크 유형:**

1. **PreToolUse** - 도구 실행 전 (검증, 경고 등)
2. **PostToolUse** - 도구 실행 후 (포매팅, 피드백 루프 등)
3. **UserPromptSubmit** - 사용자가 메시지를 보낼 때
4. **Stop** - Claude가 응답을 마쳤을 때
5. **PreCompact** - 컨텍스트 압축(Compaction) 전
6. **Notification** - 권한 요청 시

**예시: 장시간 실행되는 명령어 전 tmux 확인 경고**

```json
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm|pnpm|yarn|cargo|pytest)\"",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] 세션 유지를 위해 tmux 사용을 고려해보세요' >&2; fi"
        }
      ]
    }
  ]
}
```

![PostToolUse 후크 피드백](../../assets/images/shortform/03-posttooluse-hook.png)
*Claude Code에서 PostToolUse 후크 실행 시 나타나는 피드백 예시*

**전문가 팁:** JSON을 직접 작성하는 대신 `hookify` 플러그인을 사용하여 대화식으로 후크를 만드세요. `/hookify`를 실행하고 원하는 내용을 설명하기만 하면 됩니다.

***

## 서브 에이전트(Sub-agents)

서브 에이전트는 메인 에이전트(Orchestrator)가 작업을 위임할 수 있는 제한된 범위의 프로세스입니다. 백그라운드나 포그라운드에서 실행되며 메인 에이전트의 컨텍스트를 확보해 줍니다.

서브 에이전트는 스킬과 함께 사용할 때 강력합니다. 특정 스킬 세트를 가진 서브 에이전트에게 작업을 위임하면 자율적으로 해당 스킬을 사용해 작업을 완수합니다. 또한 특정 도구 권한만 부여하여 샌드박스화할 수도 있습니다.

```bash
# 서브 에이전트 구조 예시
~/.claude/agents/
  planner.md           # 기능 구현 계획 수립
  architect.md         # 시스템 설계 결정
  tdd-guide.md         # 테스트 주도 개발 가이드
  code-reviewer.md     # 품질 및 보안 리뷰
  security-reviewer.md # 취약점 분석
  build-error-resolver.md
  e2e-runner.md
  refactor-cleaner.md
```

각 에이전트에게 허용된 도구, MCP, 권한을 설정하여 적절한 범위를 지정해 주세요.

***

## 규칙(Rules) 및 기억(Memory)

`.rules` 폴더에는 Claude가 항상 준수해야 할 베스트 프랙티스를 담은 `.md` 파일들이 들어갑니다. 두 가지 방식이 있습니다:

1. **단일 CLAUDE.md**: 모든 내용을 하나의 파일에 작성 (사용자 또는 프로젝트 레벨)
2. **규칙 폴더(Rules Folder)**: 관심사별로 모듈화된 `.md` 파일들로 구성

```bash
~/.claude/rules/
  security.md      # 하드코딩된 비밀정보 금지, 입력값 검증 등
  coding-style.md  # 불변성, 파일 구성 방식 등
  testing.md       # TDD 워크플로우, 커버리지 80% 등
  git-workflow.md  # 커밋 형식, PR 프로세스 등
  agents.md        # 서브 에이전트 위임 기준
  performance.md   # 모델 선택, 컨텍스트 관리 등
```

**규칙 예시:**
* 코드베이스에 이모지 사용 금지
* 프런트엔드에서 보라색 계열 사용 지양
* 배포 전 항상 코드 테스트 수행
* 거대 파일보다는 모듈화된 코드 선호
* 절대로 `console.log`를 커밋하지 말 것

***

## MCP (Model Context Protocol)

MCP는 Claude를 외부 서비스에 직접 연결합니다. 단순한 API 대체제가 아니라, 정보를 탐색할 때 더 큰 유연성을 제공하는 프롬프트 기반의 API 래퍼입니다.

**예시:** Supabase MCP를 사용하면 Claude가 복사-붙여넣기 없이 직접 특정 데이터를 추출하거나 상류(upstream)에서 SQL을 실행할 수 있습니다. 데이터베이스나 배포 플랫폼 등에서도 동일하게 적용됩니다.

![Supabase MCP 테이블 목록](../../assets/images/shortform/04-supabase-mcp.jpeg)
*Supabase MCP를 사용하여 public 스키마 내 테이블을 나열하는 모습*

**Claude 내 Chrome:** 브라우저를 직접 제어할 수 있게 해주는 내장 플러그인 MCP입니다. 클릭 등을 통해 기능이 어떻게 작동하는지 직접 확인할 수 있습니다.

**핵심: 컨텍스트 윈도우 관리**
MCP 사용에는 신중해야 합니다. 저는 사용자 설정에 모든 MCP를 보관하지만, **쓰지 않는 것은 모두 비활성화**해 둡니다. `/plugins`에서 아래로 스크롤하거나 `/mcp`를 실행하여 관리하세요.

![/plugins 인터페이스 화면](../../assets/images/shortform/05-plugins-interface.jpeg)
*현재 설치된 플러그인과 상태를 확인하기 위해 /plugins를 실행한 모습*

20만 토큰의 컨텍스트 윈도우라 할지라도 너무 많은 도구가 활성화되어 있으면 실제 가용량은 7만 토큰 정도로 줄어들 수 있습니다. 이는 성능 저하로 직결됩니다.

**권장 사항:** 설정에는 20~30개의 MCP를 두되, 활성화는 10개 미만, 활성 도구는 80개 미만으로 유지하십시오.

```bash
# 활성화된 MCP 확인
/mcp

# ~/.claude.json의 projects.disabledMcpServers에서 미사용 서버 비활성화
```

***

## 플러그인(Plugins)

플러그인은 매번 수동으로 설정해야 하는 도구들을 설치하기 쉽게 패키지화한 것입니다. 스킬과 MCP의 조합일 수도 있고, 후크나 도구들의 묶음일 수도 있습니다.

**플러그인 설치:**

```bash
# 마켓플레이스 추가 예시
# mixedbread-ai의 mgrep 플러그인
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep

# Claude 실행 후 /plugins에서 마켓플레이스를 찾아 설치
```

![mgrep 마켓플레이스 탭](../../assets/images/shortform/06-marketplaces-mgrep.jpeg)
*새로 설치된 Mixedbread-Grep 마켓플레이스 화면*

**LSP 플러그인**은 IDE 밖에서 Claude Code를 자주 사용할 때 특히 유용합니다. IDE를 열지 않고도 실시간 타입 체크, 정의 이동, 자동 완성 기능을 제공합니다.

```bash
# 활성화된 플러그인 예시
typescript-lsp@claude-plugins-official  # TypeScript 지능형 기능
pyright-lsp@claude-plugins-official     # Python 타입 체크
hookify@claude-plugins-official         # 대화식 후크 생성
mgrep@Mixedbread-Grep                   # ripgrep보다 강력한 검색
```

MCP와 마찬가지로 컨텍스트 윈도우 소모에 주의하세요.

***

## 팁과 요령

### 단축키
* `Ctrl+U`: 줄 전체 삭제 (백스페이스보다 빠름)
* `!`: 빠른 bash 명령어 실행 접두사
* `@`: 파일 검색
* `/`: 슬래시 명령어 시작
* `Shift+Enter`: 여러 줄 입력
* `Tab`: 추론 과정(Thought) 표시 전환
* `Esc Esc`: Claude 중단 / 코드 복구

### 병렬 워크플로우
* **포크(Fork)**: `/fork`를 사용해 대화를 분기하여 여러 작업을 동시에 수행하세요. 메시지를 쌓아두는 것보다 빠릅니다.
* **Git Worktrees**: 충돌 없이 여러 Claude 인스턴스를 띄울 때 유용합니다. 각 워크트리는 독립된 체크아웃처럼 작동합니다.

```bash
git worktree add ../feature-branch feature-branch
# 각 워크트리에서 별도의 Claude 실행
```

### 장시간 작업용 tmux
로그나 bash 프로세스를 모니터링하며 Claude를 실행할 수 있습니다.

https://github.com/user-attachments/assets/shortform/07-tmux-video.mp4

```bash
tmux new -s dev
# 여기서 Claude 명령 실행, 세션 분리(detach) 및 재연결 가능
tmux attach -t dev
```

### mgrep > grep
`mgrep`은 ripgrep이나 grep보다 훨씬 뛰어납니다. 마켓플레이스에서 설치 후 `/mgrep` 스킬로 로컬 및 웹 검색에 활용하세요.

```bash
mgrep "function handleSubmit"  # 로컬 검색
mgrep --web "Next.js 15 app router changes"  # 웹 검색
```

### 유용한 추가 명령어
* `/rewind`: 이전 상태로 되돌리기
* `/statusline`: 브랜치, 컨테이너 정보 등으로 상태바 커스텀
* `/checkpoints`: 파일 단위의 실행 취소 지점 생성
* `/compact`: 수동으로 컨텍스트 압축 실행

### GitHub Actions CI/CD
PR에 Claude 자동 리뷰를 설정할 수 있습니다. 설정 후에는 Claude 봇이 자동으로 PR을 검토하고 의견을 남깁니다.

![Claude 봇의 PR 승인 모습](../../assets/images/shortform/08-github-pr-review.jpeg)
*버그 수정 PR을 승인하는 Claude 봇*

### 샌드박스화(Sandboxing)
리스크가 있는 작업은 샌드박스 모드로 실행하세요. 실제 시스템에 영향을 주지 않는 제한된 환경에서 작동합니다.

***

## 에디터에 관하여

어떤 에디터를 쓰느냐가 Claude Code 워크플로우에 큰 영향을 줍니다. Claude Code는 어떤 터미널에서도 작동하지만, 강력한 에디터와 함께 사용하면 실시간 파일 추적, 빠른 탐색, 통합 명령 실행 등의 이점을 누릴 수 있습니다.

### Zed (필자의 추천)
저는 [Zed](https://zed.dev)를 즐겨 사용합니다. Rust로 작성되어 정말 빠르고 리소스를 거의 차지하지 않습니다.

**Zed + Claude Code 조합이 좋은 이유:**
* **속도**: Claude가 파일을 빠르게 수정할 때 에디터가 따라오지 못하는 지연이 없습니다.
* **에이전트 패널 통합**: Claude가 작업하는 파일의 변화를 실시간으로 추적하고 해당 파일로 바로 이동할 수 있습니다.
* **CMD+Shift+R 명령 팔레트**: 커스텀 슬래시 명령어, 디버거 등을 검색 가능한 UI에서 빠르게 실행합니다.
* **최소한의 리소스**: 무거운 작업 중에도 Claude와 CPU/RAM 경쟁을 하지 않습니다. Opus 모델을 쓸 때 중요합니다.
* **Vim 모드 지원**: Vim 키 바인딩을 선호한다면 완벽합니다.

![Zed 에디터와 커스텀 명령어](../../assets/images/shortform/09-zed-editor.jpeg)
*CMD+Shift+R 키로 커스텀 명령어 메뉴를 띄운 Zed 에디터. 우측 하단의 과녁 아이콘은 팔로우 모드 활성화를 나타냅니다.*

**에디터 공통 팁:**
1. **화면 분할**: 한쪽엔 Claude Code 터미널, 다른 쪽엔 에디터를 띄우세요.
2. **Ctrl + G**: Zed에서 Claude가 현재 작업 중인 파일로 빠르게 이동합니다.
3. **자동 저장**: 에디터의 자동 저장 기능을 켜서 Claude가 항상 최신 파일 내용을 읽게 하세요.
4. **Git 통합**: 커밋 전 에디터의 Git 기능을 이용해 Claude가 수정한 내용을 최종 검토하세요.

### VSCode / Cursor
전통적이고 훌륭한 선택지입니다. 터미널 포맷이나 `\ide` 명령을 통해 LSP 기능을 연동할 수도 있고, 전용 확장 프로그램(Extension)을 사용해 더 통합된 UI를 누릴 수도 있습니다.

![VS Code용 Claude Code 확장 프로그램](../../assets/images/shortform/10-vscode-extension.jpeg)
*VS Code 확장은 IDE 내부에 통합된 그래픽 인터페이스를 제공합니다.*

***

## 저의 설정 구성

### 플러그인 목록
(보통 한 번에 4~5개만 활성화해서 사용합니다)
```markdown
ralph-wiggum@claude-code-plugins       # 자동화 루프
frontend-design@claude-code-plugins    # UI/UX 패턴
commit-commands@claude-code-plugins    # Git 워크플로우
security-guidance@claude-code-plugins  # 보안 점검
pr-review-toolkit@claude-code-plugins  # PR 자동화
typescript-lsp@claude-plugins-official # TS 지능적 기능
hookify@claude-plugins-official        # 후크 생성 보조
context7@claude-plugins-official       # 실시간 문서화
mgrep@Mixedbread-Grep                   # 더 나은 검색
```

### MCP 서버 설정 (사용자 레벨)
```json
{
  "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"] },
  "firecrawl": { "command": "npx", "args": ["-y", "firecrawl-mcp"] },
  "supabase": { "command": "npx", "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=YOUR_REF"] },
  "memory": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-memory"] },
  "sequential-thinking": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"] },
  "vercel": { "type": "http", "url": "https://mcp.vercel.com" },
  "railway": { "command": "npx", "args": ["-y", "@railway/mcp-server"] },
  "clickhouse": { "type": "http", "url": "https://mcp.clickhouse.cloud/mcp" },
  "magic": { "command": "npx", "args": ["-y", "@magicuidesign/mcp@latest"] }
}
```
핵심은 **설정은 많되, 프로젝트별 활성화는 5~6개로 제한**하여 컨텍스트 가용량을 지키는 것입니다.

### 주요 후크 설정
```json
{
  "PreToolUse": [
    { "matcher": "npm|pnpm|yarn|cargo|pytest", "hooks": ["tmux 확인 경고"] },
    { "matcher": "Write && .md 파일", "hooks": ["README/CLAUDE 제외 차단"] },
    { "matcher": "git push", "hooks": ["리뷰용 에디터 열기"] }
  ],
  "PostToolUse": [
    { "matcher": "Edit && .ts/.tsx/.js/.jsx", "hooks": ["prettier --write"] },
    { "matcher": "Edit && .ts/.tsx", "hooks": ["tsc --noEmit"] },
    { "matcher": "Edit", "hooks": ["console.log 경고 탐색"] }
  ],
  "Stop": [
    { "matcher": "*", "hooks": ["수정된 파일 내 console.log 최종 확인"] }
  ]
}
```

### 커스텀 상태 라인 (Statusline)
브랜치, 컨텍스트 잔량, 모델, 시간, 할 일 목록 등을 표시합니다:

![커스텀 상태바 레이아웃](../../assets/images/shortform/11-statusline.jpeg)

```
affoon:~ ctx:65% Opus 4.5 19:52
▌▌ plan mode on (shift+tab to cycle)
```

***

## 핵심 요약
1. **지나치게 복잡하게 만들지 마세요** - 설정은 '미세 조정'이지 건축이 아닙니다.
2. **컨텍스트 윈도우는 소중합니다** - 미사용 MCP와 플러그인은 끄세요.
3. **병렬로 작업하세요** - 대화 포크와 git worktree를 활용하세요.
4. **반복 작업은 자동화하세요** - 포매팅, 린팅, 경고 등에 후크를 쓰세요.
5. **서브 에이전트의 범위를 좁히세요** - 제한된 도구가 집중력을 높입니다.

***

## 참고 자료
* [플러그인 레퍼런스](https://code.claude.com/docs/en/plugins-reference)
* [후크 가이드](https://code.claude.com/docs/en/hooks)
* [체크포인트 시스템](https://code.claude.com/docs/en/checkpointing)
* [인터랙티브 모드](https://code.claude.com/docs/en/interactive-mode)
* [기억(Memory) 시스템](https://code.claude.com/docs/en/memory)
* [서브 에이전트 가이드](https://code.claude.com/docs/en/sub-agents)
* [MCP 개요](https://code.claude.com/docs/en/mcp-overview)

***

더 상세하고 고급화된 패턴은 [상세 가이드](the-longform-guide.md)를 참조하십시오.

***

*뉴욕에서 [@DRodriguezFX](https://x.com/DRodriguezFX)와 함께 [zenith.chat](https://zenith.chat)을 개발하여 Anthropic x Forum Ventures 해커톤에서 우승했습니다.*
