# Claude Code에 관한 종합 상세 가이드 (The Longform Guide)

![Header: The Longform Guide to Everything Claude Code](../../assets/images/longform/01-header.png)

***

> **전제 조건**: 이 가이드는 [Claude Code 요약 가이드](the-shortform-guide.md)의 내용을 바탕으로 합니다. 스킬, 후크, 서브 에이전트, MCP 및 플러그인을 아직 설정하지 않았다면 먼저 해당 가이드를 읽어보시기 바랍니다.

![Reference to Shorthand Guide](../../assets/images/longform/02-shortform-reference.png)
*요약 가이드 - 먼저 읽어보세요*

요약 가이드에서는 스킬과 명령어, 후크, 서브 에이전트, MCP, 플러그인 등 효율적인 Claude Code 워크플로우를 구성하는 기초 설정과 아키텍처를 다루었습니다.

이 상세 가이드에서는 생산적인 세션과 낭비되는 세션을 가르는 고급 기술들을 깊이 있게 다룹니다. 요약 가이드를 읽지 않았다면 먼저 돌아가서 설정을 완료하십시오. 본 내용은 이미 스킬, 에이전트, 후크, MCP가 구성되어 정상 작동하고 있음을 가정합니다.

여기서 다룰 주제는 토큰 경제, 메모리 유지, 검증 패턴, 병렬화 전략, 그리고 재사용 가능한 워크플로우 구축의 복합 효과입니다. 10개월 이상의 일상적인 사용을 통해 정립된 이 패턴들은, 세션 시작 1시간 만에 컨텍스트 오염으로 고생하느냐, 아니면 수 시간 동안 높은 생산성을 유지하느냐를 결정짓는 핵심 요소입니다.

요약 가이드와 상세 가이드에서 다루는 모든 내용은 GitHub에서 찾을 수 있습니다: `github.com/affaan-m/everything-claude-code`

***

## 팁과 요령

### 일부 MCP는 대체 가능하며 컨텍스트 윈도우를 확보해 줍니다

버전 관리(GitHub), 데이터베이스(Supabase), 배포(Vercel, Railway)와 같은 MCP들의 경우, 대부분 이미 강력한 CLI를 보유하고 있으며 MCP는 본질적으로 이를 래핑(Wrapping)한 것에 불과합니다. MCP는 편리한 래퍼이지만 비용이 따릅니다.

CLI 기능을 MCP처럼 사용하면서 실제 MCP를 사용하지 않고(따라서 컨텍스트 윈도우를 소모하지 않고) 기능을 활용하려면, 해당 기능을 스킬과 명령어로 패키징하는 것을 고려하십시오. MCP가 노출하는 편리한 도구들을 추출하여 명령어로 변환하십시오.

예: 항상 GitHub MCP를 로드하는 대신, 선호하는 옵션이 포함된 `gh pr create`를 실행하는 `/gh-pr` 명령어를 만드십시오. Supabase MCP가 컨텍스트를 소모하게 두는 대신, Supabase CLI를 직접 사용하는 스킬을 만드십시오.

지연 로딩(Lazy loading) 덕분에 컨텍스트 윈도우 문제는 어느 정도 해결되었지만, 토큰 사용량과 비용 문제는 여전히 남아 있습니다. CLI + 스킬 접근 방식은 여전히 유효한 토큰 최적화 방법입니다.

***

## 주요 사항

### 컨텍스트 및 메모리 관리

세션 간에 메모리를 공유하는 가장 좋은 방법은 스킬이나 명령어를 사용하여 진행 상황을 요약 및 확인하고, 이를 `.claude` 폴더 내의 `.tmp` 파일에 저장한 뒤 세션 종료 전까지 계속 내용을 추가하는 것입니다. 다음 날 이를 컨텍스트로 사용하여 중단된 지점부터 다시 시작할 수 있습니다. 각 세션마다 새 파일을 생성하여 이전 컨텍스트가 새 작업에 섞이지 않도록 하십시오. 이 파일에는 다음 내용이 포함되어야 합니다:

* 어떤 시도가 성공했는가 (검증 가능한 증거 포함)
* 어떤 시도를 했으나 실패했는가
* 아직 시도하지 않은 것은 무엇이며, 무엇이 남았는가

![Session Storage File Tree](../../assets/images/longform/03-session-storage.png)
*세션 저장 예시 -> https://github.com/affaan-m/everything-claude-code/tree/main/examples/sessions*

Claude에게 현재 상태를 요약하는 파일을 만들게 하십시오. 이를 검토하고 필요한 경우 수정을 요청한 뒤 새로 시작하십시오. 새 대화에서는 해당 파일 경로만 제공하면 됩니다. 컨텍스트 한도에 도달하여 복잡한 작업을 계속해야 할 때 특히 유용합니다.

**전략적 컨텍스트 초기화:**

계획을 수립하고 컨텍스트를 초기화한 후(Claude Code 계획 모드의 기본 옵션), 해당 계획에 따라 작업을 진행하십시오. 실행과 관련 없는 탐색적 컨텍스트가 너무 많이 쌓였을 때 유용합니다. 전략적 압축을 위해 자동 압축(auto-compaction)을 비활성화하고, 논리적인 단계마다 수동으로 압축하거나 이를 대신 수행해 주는 스킬을 사용하십시오.

**고급: 동적 시스템 프롬프트 주입**

모든 내용을 CLAUDE.md(사용자 범위)나 `.claude/rules/`(프로젝트 범위)에 넣어 매 세션마다 로드하는 대신, CLI 플래그를 사용하여 컨텍스트를 동적으로 주입하는 패턴입니다.

```bash
claude --system-prompt "$(cat memory.md)"
```

이를 통해 어떤 컨텍스트를 언제 로드할지 더 정밀하게 제어할 수 있습니다. 시스템 프롬프트는 사용자 메시지보다, 사용자 메시지는 도구 결과보다 더 높은 우선순위와 권위를 가집니다.

**실제 설정 예시:**

```bash
# 일상적인 개발용
alias claude-dev='claude --system-prompt "$(cat ~/.claude/contexts/dev.md)"'

# PR 리뷰 모드
alias claude-review='claude --system-prompt "$(cat ~/.claude/contexts/review.md)"'

# 연구/탐색 모드
alias claude-research='claude --system-prompt "$(cat ~/.claude/contexts/research.md)"'
```

**고급: 메모리 유지 후크**

많은 사람들이 모르고 있는 메모리 관리용 후크가 있습니다:

* **PreCompact 후크**: 컨텍스트 압축이 발생하기 직전, 중요한 상태를 파일로 저장
* **Stop 후크 (세션 종료)**: 세션 종료 시 학습된 내용을 파일로 영속화
* **SessionStart 후크**: 새 세션 시작 시 이전 컨텍스트를 자동 로드

이러한 후크들을 구현하여 저장소의 `github.com/affaan-m/everything-claude-code/tree/main/hooks/memory-persistence`에 올려두었습니다.

***

### 지속적 학습 / 메모리

특정 프롬프트를 여러 번 반복해야 하거나, Claude가 이전에 겪었던 문제와 똑같은 문제에 부딪히는 경우 해당 패턴을 반드시 스킬에 추가해야 합니다.

**문제점:** 토큰 낭비, 컨텍스트 낭비, 시간 낭비.

**해결책:** Claude Code가 디버깅 요령, 우회 방법, 특정 프로젝트 패턴 등 중요한 사항을 발견하면 이를 새로운 스킬로 저장합니다. 다음에 유사한 문제가 발생하면 해당 스킬이 자동으로 로드됩니다.

이를 구현한 지속적 학습 스킬: `github.com/affaan-m/everything-claude-code/tree/main/skills/continuous-learning`

**왜 Stop 후크인가? (UserPromptSubmit 대신):**

핵심 설계 결정은 UserPromptSubmit 대신 **Stop 후크**를 사용하는 것입니다. UserPromptSubmit은 모든 메시지마다 실행되어 지연시간을 유발하지만, Stop은 세션 종료 시 한 번만 실행되므로 가볍고 세션 진행 속도를 늦추지 않습니다.

***

### 토큰 최적화

**주요 전략: 서브 에이전트 아키텍처**

작업을 가장 저렴하면서도 충분히 수행 가능한 모델에 위임하도록 서브 에이전트 아키텍처를 최적화하십시오.

**모델 선택 가이드:**

![Model Selection Table](../../assets/images/longform/04-model-selection.png)
*다양한 작업에 따른 하위 에이전트 설정 예시 및 이유*

| 작업 유형 | 모델 | 이유 |
|-----------|------|------|
| 탐색/검색 | Haiku | 파일 검색에 충분히 빠르고 저렴함 |
| 단순 편집 | Haiku | 명확한 지침이 있는 단일 파일 수정 |
| 다중 파일 구현 | Sonnet | 코딩을 위한 최적의 균형 |
| 복잡한 아키텍처 | Opus | 깊은 추론이 필요함 |
| PR 리뷰 | Sonnet | 컨텍스트 이해 및 미묘한 차이 포착 |
| 보안 분석 | Opus | 취약점을 놓쳐서는 안 됨 |
| 문서 작성 | Haiku | 단순한 구조 작업 |
| 복잡한 오류 디버깅 | Opus | 시스템 전체를 파악해야 함 |

90%의 코딩 작업에는 Sonnet을 기본으로 사용하십시오. 첫 번째 시도가 실패하거나, 5개 이상의 파일이 관련되거나, 아키텍처 결정 또는 보안이 중요한 코드인 경우 Opus로 업그레이드하십시오.

**도구별 최적화:**

grep을 mgrep으로 교체하면 기존 grep이나 ripgrep 대비 평균 약 50%의 토큰을 절약할 수 있습니다.

![mgrep Benchmark](../../assets/images/longform/06-mgrep-benchmark.png)
*50개 작업 벤치마크 결과, mgrep + Claude Code 조합이 기존 grep 방식보다 토큰을 약 2배 적게 사용하면서도 동일하거나 더 나은 품질을 보여주었습니다. 출처: @mixedbread-ai의 mgrep*

**모듈화된 코드베이스의 장점:**

대형 파일 하나로 구성된 것이 아니라 본문이 수백 라인 단위인 모듈화된 코드베이스를 유지하면 토큰 최적화 비용을 낮출 수 있고, 첫 시도에 성공할 확률도 높아집니다.

***

### 검증 루프 및 평가 (Evals)

**벤치마크 워크플로우:**

동일한 요청을 스킬이 있을 때와 없을 때 각각 수행하여 출력 차이를 비교해 보십시오. 대화를 분리(fork)하여 한쪽에서는 스킬을 사용하지 않고 시도한 뒤 차이점(diff)을 확인하여 무엇이 개선되었는지 기록하십시오.

**평가 모드 유형:**

* **체크포인트 기반 평가**: 명확한 체크포인트를 설정하고, 정의된 기준에 따라 검증한 뒤 수정 후 다음 단계로 진행
* **지속적 평가**: N분마다 또는 대규모 변경 후 전체 테스트 슈트 및 Lint 실행

**핵심 지표:**

```
pass@k: k번의 시도 중 최소 한 번 성공할 확률
        k=1: 70%  k=3: 91%  k=5: 97%

pass^k: k번의 시도 모두 성공해야 함
        k=1: 70%  k=3: 34%  k=5: 17%
```

작동하기만 하면 될 때는 **pass@k**를, 일관성이 중요할 때는 **pass^k**를 기준으로 삼으십시오.

***

## 병렬화 (Parallelization)

여러 Claude 터미널에서 대화를 분리할 때, 분리된 세션과 원본 세션의 역할 범위를 명확히 정의하십시오. 코드 변경 시 작업이 겹치지 않도록 해야 합니다.

**권장 패턴:**

메인 채팅은 코드 변경에 사용하고, 분리된 채팅은 코드베이스의 상태 질문이나 외부 서비스 조사용으로 사용하십시오.

**터미널 개수에 관하여:**

![Boris on Parallel Terminals](../../assets/images/longform/07-boris-parallel.png)
*여러 Claude 인스턴스 실행에 관한 Boris(Anthropic)의 코멘트*

단순히 터미널 개수를 늘리는 것이 목표가 되어서는 안 됩니다. 터미널 추가는 실제 필요에 의해 이루어져야 합니다. 목표는 **필요한 최소한의 병렬화로 최대한의 작업량을 소화하는 것**이어야 합니다.

**병렬 인스턴스를 위한 Git Worktrees:**

```bash
# 병렬 작업을 위한 워크트리 생성
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
git worktree add ../project-refactor refactor-branch

# 각 워크트리마다 독립적인 Claude 인스턴스 실행
cd ../project-feature-a && claude
```

여러 인스턴스가 서로 겹치는 코드를 수정해야 한다면 반드시 git worktrees를 사용하고, 각 인스턴스에 매우 명확한 계획을 부여하십시오. `/rename <이름>`을 사용하여 채팅 세션에 이름을 붙이십시오.

![Two Terminal Setup](../../assets/images/longform/08-two-terminals.png)
*초기 설정: 왼쪽 터미널은 코딩, 오른쪽 터미널은 질문용 - /rename 및 /fork 명령어 활용*

**계층적 접근 방식 (Cascading Method):**

여러 Claude Code 인스턴스를 실행할 때 다음 방식으로 조율하십시오:

* 오른쪽 새 탭에서 새 작업을 시작
* 왼쪽에서 오른쪽으로, 오래된 것에서 최신 순으로 스캔
* 한 번에 최대 3~4개의 작업에만 집중

***

## 기초 작업 (Foundations)

**이중 인스턴스 시작 모드:**

워크플로우 관리를 위해 빈 저장소에서 2개의 Claude 인스턴스로 시작하는 것을 선호합니다.

**인스턴스 1: 스켈레톤 에이전트 (Scaffold Agent)**

* 구조 잡기 및 기초 작업 수행
* 프로젝트 구조 생성
* 설정(CLAUDE.md, 규칙, 에이전트) 구성

**인스턴스 2: 심층 연구 에이전트 (Deep Research Agent)**

* 모든 서비스 연결 및 웹 검색 수행
* 상세 PRD 작성
* 아키텍처 Mermaid 다이어그램 작성
* 실제 문서 스니펫이 포함된 참조 자료 수집

**llms.txt 패턴:**

문서 페이지에 도달했을 때 `/llms.txt`를 실행하여 해당 사이트의 `llms.txt`를 찾아보십시오. LLM에 최적화된 깨끗한 버전의 문서를 얻을 수 있습니다.

**철학: 재사용 가능한 패턴 구축**

@omarsar0의 한마디: "초기에는 재사용 가능한 워크플로우/패턴을 구축하는 데 시간을 보냈습니다. 지루한 과정이었지만 모델과 에이전트 프레임워크가 개선됨에 따라 이는 엄청난 복합 효과를 가져왔습니다."

**투자해야 할 가치가 있는 것:**

* 서브 에이전트
* 스킬
* 명령어
* 계획 수립 패턴
* MCP 도구 및 컨텍스트 엔지니어링 패턴

***

## 에이전트 및 서브 에이전트 최선 관행

**서브 에이전트 컨텍스트 문제:**

서브 에이전트는 모든 내용을 쏟아내는 대신 요약을 반환하여 컨텍스트를 절약하기 위해 존재합니다. 하지만 오케스트레이터(메인 에이전트)는 서브 에이전트가 모르는 의미론적 컨텍스트를 가지고 있습니다. 서브 에이전트는 쿼리 자체는 알지만 그 이면의 **목적**을 모를 수 있습니다.

**반복적 검색 패턴:**

1. 오케스트레이터가 각 서브 에이전트의 결과 평가
2. 결과를 수용하기 전 후속 질문 던지기
3. 서브 에이전트가 출처 확인 후 답변 수정 및 반환
4. 충분할 때까지 반복 (최대 3회 권장)

**핵심:** 단순 쿼리뿐만 아니라 목표 컨텍스트를 함께 전달하십시오.

**순차적 단계가 있는 오케스트레이션:**

```markdown
1단계: 연구 (탐색 에이전트 활용) → research-summary.md
2단계: 계획 (계획 에이전트 활용) → plan.md
3단계: 구현 (TDD 가이드 에이전트 활용) → 코드 변경
4단계: 리뷰 (코드 리뷰 에이전트 활용) → review-comments.md
5단계: 검증 (필요시 빌드 에러 해결사 활용) → 완료 또는 루프 재시작
```

**핵심 규칙:**

1. 각 에이전트는 명확한 입력과 출력을 가짐
2. 이전 단계의 출력이 다음 단계의 입력이 됨
3. 단계를 절대 건너뛰지 않음
4. 에이전트 교체 시 `/clear` 사용
5. 중간 출력물은 파일에 저장

***

## 흥미로운 팁 (비핵심적)

### 사용자 정의 상태 표시줄 (Statusline)

`/statusline` 명령어로 설정할 수 있습니다. Claude가 도구가 없다고 답하면 설정해 달라고 요청하고 무엇을 넣을지 말하십시오. (커뮤니티 프로젝트인 ccstatusline 참조)

### 음성 받아쓰기

Claude Code와 음성으로 대화하십시오. 타이핑보다 훨씬 빠를 수 있습니다. (Mac의 superwhisper, MacWhisper 등 활용) 전사가 완벽하지 않아도 Claude는 의도를 파악합니다.

### 터미널 별칭 (Alias)

```bash
alias c='claude'
alias gb='github'
alias co='code'
alias q='cd ~/Desktop/projects'
```

***

## 마일스톤

![25k+ GitHub Stars](../../assets/images/longform/09-25k-stars.png)
*일주일 만에 25,000개 이상의 GitHub 별(stars) 획득*

***

## 리소스

**에이전트 오케스트레이션:**
* claude-flow — 54개 이상의 전문 에이전트를 포함한 커뮤니티 구축 엔터프라이즈 오케스트레이션 플랫폼

**자기 개선 메모리:**
* 본 저장소의 `skills/continuous-learning/` 참조
* rlancemartin.github.io/2025/12/01/claude_diary/ - 세션 회고 패턴

**시스템 프롬프트 참조:**
* system-prompts-and-models-of-ai-tools — 커뮤니티 수집 AI 시스템 프롬프트 모음

**공식 채널:**
* Anthropic Academy: anthropic.skilljar.com

***

## 참고 자료

* [Anthropic: 에이전트 평가의 비밀](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
* [YK: Claude Code 팁 32선](https://agenticcoding.substack.com/p/32-claude-code-tips-from-basics-to)
* [RLanceMartin: 세션 회고 패턴](https://rlancemartin.github.io/2025/12/01/claude_diary/)
* @PerceptualPeak: 서브 에이전트 컨텍스트 협상
* @menhguin: 에이전트 추상화 수준
* @omarsar0: 복합 효과의 철학

***

*두 가이드에서 다루는 모든 내용은 GitHub의 [everything-claude-code](https://github.com/affaan-m/everything-claude-code)에서 확인할 수 있습니다.*
