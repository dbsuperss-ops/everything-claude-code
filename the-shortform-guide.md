# Everything Claude Code 단축 가이드 (The Shorthand Guide)

![Header: Anthropic 해커톤 우승자 - Claude Code를 위한 팁과 요령](./assets/images/shortform/00-header.png)

---

**2월 실험적 출시 이후 Claude Code를 열성적으로 사용해 왔으며, [@DRodriguezFX](https://x.com/DRodriguezFX)와 함께 [zenith.chat](https://zenith.chat)을 구축하여 Anthropic x Forum Ventures 해커톤에서 우승했습니다 - 전적으로 Claude Code만을 사용해서 말이죠.**

10개월간의 일상적인 사용을 통해 완성된 저의 전체 설정(스킬, 후크, 서브 에이전트, MCP, 플러그인 등)과 실제로 효과가 있는 방법들을 공유합니다.

---

## 스킬 및 명령어 (Skills and Commands)

스킬은 특정 범위와 워크플로우로 제한된 규칙처럼 작동합니다. 특정 워크플로우를 실행해야 할 때 프롬프트를 대신하는 단축키와 같습니다.

Opus 4.5와 긴 코딩 세션을 마친 후, 사용하지 않는 코드나 불필요한 .md 파일을 정리하고 싶으신가요? `/refactor-clean`을 실행하십시오. 테스트가 필요하다면? `/tdd`, `/e2e`, `/test-coverage`를 사용하십시오. 스킬에는 코드맵(codemaps)도 포함될 수 있는데, 이는 Claude가 탐색에 컨텍스트를 낭비하지 않고 코드베이스를 빠르게 탐색할 수 있는 방법입니다.

![체인된 명령어를 보여주는 터미널](./assets/images/shortform/02-chaining-commands.jpeg)
*명령어 체이닝(Chaining)*

명령어는 슬래시 명령어를 통해 실행되는 스킬입니다. 기능은 겹치지만 저장 위치가 다릅니다:

- **스킬 (Skills)**: `~/.claude/skills/` - 광범위한 워크플로우 정의
- **명령어 (Commands)**: `~/.claude/commands/` - 빠른 실행 프롬프트

```bash
# 예시 스킬 구조
~/.claude/skills/
  pmx-guidelines.md      # 프로젝트별 패턴
  coding-standards.md    # 언어별 최선 관행
  tdd-workflow/          # README.md를 포함한 다중 파일 스킬
  security-review/       # 체크리스트 기반 스킬
```

---

## 후크 (Hooks)

후크는 특정 이벤트가 발생할 때 실행되는 트리거 기반 자동화입니다. 스킬과 달리 도구 호출 및 라이프사이클 이벤트로 제한됩니다.

**후크 유형:**

1. **PreToolUse** - 도구 실행 전 (검증, 리마인더)
2. **PostToolUse** - 도구 종료 후 (포맷팅, 피드백 루프)
3. **UserPromptSubmit** - 사용자가 메시지를 보낼 때
4. **Stop** - Claude가 응답을 마쳤을 때
5. **PreCompact** - 컨텍스트 압축 전
6. **Notification** - 권한 요청 시

**예시: 실행 시간이 긴 명령 전 tmux 리마인더**

```json
{
  "PreToolUse": [
    {
      "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm|pnpm|yarn|cargo|pytest)\"",
      "hooks": [
        {
          "type": "command",
          "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] 세션 유지를 위해 tmux 사용을 고려해 보세요' >&2; fi"
        }
      ]
    }
  ]
}
```

![PostToolUse 후크 피드백](./assets/images/shortform/03-posttooluse-hook.png)
*Claude Code에서 PostToolUse 후크가 실행될 때 나타나는 피드백 예시*

**프로 팁:** `hookify` 플러그인을 사용하여 JSON을 직접 작성하는 대신 대화식으로 후크를 만드십시오. `/hookify`를 실행하고 원하는 내용을 설명하면 됩니다.

---

## 서브 에이전트 (Subagents)

서브 에이전트는 오케스트레이터(메인 Claude)가 제한된 범위의 작업을 위임할 수 있는 프로세스입니다. 백그라운드나 포그라운드에서 실행될 수 있으며, 메인 에이전트의 컨텍스트를 확보해 줍니다.

서브 에이전트는 스킬과 잘 상호작용합니다 - 사용자의 스킬 중 일부를 실행할 수 있는 서브 에이전트에게 작업을 위임하고 해당 스킬을 자율적으로 사용하게 할 수 있습니다. 또한 특정 도구 권한으로 샌드박싱될 수도 있습니다.

```bash
# 예시 서브 에이전트 구조
~/.claude/agents/
  planner.md           # 기능 구현 계획
  architect.md         # 시스템 설계 결정
  tdd-guide.md         # 테스트 주도 개발
  code-reviewer.md     # 품질/보안 리뷰
  security-reviewer.md # 취약점 분석
  build-error-resolver.md
  e2e-runner.md
  refactor-cleaner.md
```

적절한 범위 지정을 위해 서브 에이전트별로 허용된 도구, MCP 및 권한을 구성하십시오.

---

## 규칙 및 메모리 (Rules and Memory)

`.rules` 폴더에는 Claude가 항상 준수해야 할 최선 관행이 담긴 `.md` 파일들이 포함됩니다. 두 가지 접근 방식이 있습니다:

1. **단일 CLAUDE.md** - 모든 내용을 하나의 파일에 작성 (사용자 또는 프로젝트 수준)
2. **Rules 폴더** - 관심사별로 그룹화된 모듈형 `.md` 파일들

```bash
~/.claude/rules/
  security.md      # 보안 규칙 (비밀값 금지, 입력 검증)
  coding-style.md  # 코딩 스타일 (불변성, 파일 구성)
  testing.md       # 테스트 규칙 (TDD 워크플로우, 커버리지 80%)
  git-workflow.md  # Git 워크플로우 (커밋 포맷, PR 프로세스)
  agents.md        # 에이전트 규칙 (서브 에이전트 위임 시점)
  performance.md   # 성능 규칙 (모델 선택, 컨텍스트 관리)
```

**예시 규칙:**
- 코드베이스에서 이모지 사용 금지
- 프런트엔드에서 보라색 톤 사용 지양
- 배포 전 항상 코드 테스트 수행
- 큰 파일 하나보다 모듈화된 코드 우선
- console.log 커밋 금지

---

## MCP (Model Context Protocol)

MCP는 Claude를 외부 서비스에 직접 연결합니다. API를 대체하는 것이 아니라, 정보를 탐색하는 데 더 높은 유연성을 제공하는 프롬프트 기반 래퍼입니다.

**예시:** Supabase MCP를 사용하면 Claude가 복사-붙여넣기 없이 직접 특정 데이터를 가져오거나 업스트림에서 SQL을 실행할 수 있습니다. 데이터베이스, 배포 플랫폼 등에서도 마찬가지입니다.

![Supabase MCP 테이블 나열](./assets/images/shortform/04-supabase-mcp.jpeg)
*공용 스키마 내의 테이블을 나열하는 Supabase MCP 예시*

**Claude 내 Chrome:** 브라우저를 자율적으로 제어하여 동작을 확인하는 빌트인 플러그인 MCP입니다.

**중요: 컨텍스트 윈도우 관리**

MCP 선택에 신중하십시오. 저는 모든 MCP를 사용자 설정에 두되, **사용하지 않는 것은 모두 비활성화**합니다. `/plugins`에서 아래로 스크롤하거나 `/mcp`를 실행하여 확인하십시오.

![/plugins 인터페이스](./assets/images/shortform/05-plugins-interface.jpeg)
*압축 전 200k 컨텍스트 윈도우가 너무 많은 도구를 활성화하면 70k로 줄어들 수 있습니다. 이 경우 성능이 눈에 띄게 저하됩니다.*

**기본 원칙:** 설정에는 20~30개의 MCP를 두되, 활성화된 것은 10개 미만 / 활성화된 도구는 80개 미만으로 유지하십시오.

---

## 플러그인 (Plugins)

플러그인은 번거로운 수동 설정 대신 도구들을 패키지화하여 쉽게 설치할 수 있게 해줍니다. 플러그인은 스킬 + MCP의 결합체이거나 후크/도구들의 묶음일 수 있습니다.

**플러그인 설치:**

```bash
# 마켓플레이스 추가
# @mixedbread-ai의 mgrep 플러그인
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep

# Claude를 열고 /plugins를 실행하여 새 마켓플레이스를 찾아 설치
```

**LSP 플러그인**은 에디터 외부에서 Claude Code를 자주 실행할 때 특히 유용합니다. Language Server Protocol은 IDE를 열지 않고도 Claude에게 실시간 타입 체크, 정의로 이동, 지능형 완성을 제공합니다.

MCP와 마찬가지로 컨텍스트 윈도우 소모에 주의하십시오.

---

## 팁과 요령

### 단축키

- `Ctrl+U` - 라인 전체 삭제 (백스페이스 연타보다 빠름)
- `!` - 빠른 Bash 명령 접두사
- `@` - 파일 검색
- `/` - 슬래시 명령어 시작
- `Shift+Enter` - 멀티라인 입력
- `Tab` - 추론(thinking) 표시 전환
- `Esc Esc` - Claude 중단 / 코드 복구

### 병렬 워크플로우

- **Fork** (`/fork`) - 대기 중인 메시지를 양산하는 대신 대화를 포크하여 겹치지 않는 작업을 병렬로 수행
- **Git Worktrees** - 충돌 없이 겹치는 작업을 병렬로 수행하기 위해 사용. 각 워크트리는 독립적인 체크아웃입니다.

### 긴 작업에는 tmux 추천

Claude가 실행하는 Bash 프로세스나 로그를 스트리밍하고 지켜보세요.

```bash
tmux new -s dev
# Claude가 여기서 명령을 실행하며, 사용자는 연결을 끊었다가 다시 붙일 수 있습니다.
tmux attach -t dev
```

### mgrep > grep

`mgrep`은 ripgrep/grep보다 크게 개선된 도구입니다. 플러그인 마켓플레이스를 통해 설치하고 `/mgrep` 스킬을 사용하십시오. 로컬 검색과 웹 검색 모두 지원합니다.

### 기타 유용한 명령어

- `/rewind` - 이전 상태로 되돌리기
- `/statusline` - 브랜치, 컨텍스트 %, 할 일 등으로 커스텀
- `/checkpoints` - 파일 수준의 실행 취소 지점
- `/compact` - 컨텍스트 압축 수동 실행

---

## 에디터에 관하여

에디터 선택은 Claude Code 워크플로우에 큰 영향을 미칩니다. Claude Code는 모든 터미널에서 작동하지만, 유능한 에디터와 결합하면 실시간 파일 추적, 빠른 탐색 및 통합 명령 실행이 가능해집니다.

### Zed (필자의 선호)

저는 Rust로 작성되어 정말 빠른 [Zed](https://zed.dev)를 사용합니다. 즉시 열리고, 거대한 코드베이스도 끊김 없이 처리하며 시스템 리소스를 거의 차지하지 않습니다.

**Zed + Claude Code 조합이 좋은 이유:**
- **속도** - Claude가 파일을 빠르게 편집할 때 지연 시간이 없습니다.
- **에이전트 패널 통합** - Claude가 파일을 편집할 때 실시간으로 변경 사항을 추적할 수 있습니다.
- **커맨드 팔레트 (CMD+Shift+R)** - 커스텀 슬래시 명령어, 디버거, 빌드 스크립트에 빠르게 접근할 수 있습니다.
- **최소 리소스 사용** - Opus 실행 시 등 무거운 작업 중에 RAM/CPU를 Claude와 경쟁하지 않습니다.
- **Vim 모드** - 필요한 경우 완전한 Vim 키바인딩을 지원합니다.

### VSCode / Cursor

이 역시 좋은 선택이며 Claude Code와 잘 작동합니다. `\ide`를 사용하여 LSP 기능을 활성화하거나(이제 플러그인으로 인해 다소 중복됨), 에디터와 더 긴밀하게 통합된 확장 프로그램을 선택할 수 있습니다.

---

## 저의 설정 (My Setup)

### 플러그인
저는 보통 한 번에 4~5개만 활성화하여 사용합니다 (typescript-lsp, hookify, mgrep 등).

### MCP 서버
14개의 MCP가 구성되어 있지만 프로젝트당 5~6개만 활성화하여 컨텍스트 윈도우를 건강하게 유지합니다.

### 핵심 후크
필요한 작업(PreToolUse, PostToolUse, Stop 등)에 대해 tmux 리마인더, 파일 작성 제한, Prettier 자동 실행, console.log 경고 등을 설정해 두었습니다.

---

## 핵심 요약

1. **지나치게 복잡하게 만들지 마십시오** - 설정을 아키텍처가 아닌 미세 조정(fine-tuning)처럼 다루십시오.
2. **컨텍스트 윈도우는 소중합니다** - 사용하지 않는 MCP와 플러그인은 비활성화하십시오.
3. **병렬 실행을 활용하십시오** - 대화를 포크하고 Git worktrees를 사용하십시오.
4. **반복적인 일은 자동화하십시오** - 포맷팅, 린팅, 리마인더를 위해 후크를 사용하십시오.
5. **서브 에이전트의 범위를 지정하십시오** - 제한된 도구는 집중된 실행을 의미합니다.

---

*NYC에서 개최된 Anthropic x Forum Ventures 해커톤에서 [@DRodriguezFX](https://x.com/DRodriguezFX)와 함께 [zenith.chat](https://zenith.chat)을 구축하여 우승했습니다.*
