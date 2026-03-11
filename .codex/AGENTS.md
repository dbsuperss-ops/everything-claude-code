# Codex CLI용 ECC (ECC for Codex CLI)

이 문서는 루트 디렉토리의 `AGENTS.md`를 보완하며, Codex 특화 가이드를 제공합니다.

## 모델 권장 사항 (Model Recommendations)

| 작업 유형 | 권장 모델 |
|-----------|------------------|
| 루틴 코딩, 테스트, 포매팅 | GPT 5.4 |
| 복잡한 기능, 아키텍처 | GPT 5.4 |
| 디버깅, 리팩토링 | GPT 5.4 |
| 보안 리뷰 | GPT 5.4 |

## 스킬 탐색 (Skills Discovery)

스킬은 `.agents/skills/`에서 자동으로 로드됩니다. 각 스킬은 다음을 포함합니다:
- `SKILL.md` — 상세 지침 및 워크플로우
- `agents/openai.yaml` — Codex 인터페이스 메타데이터

사용 가능한 스킬:
- tdd-workflow — 80% 이상의 커버리지를 목표로 하는 테스트 주도 개발
- security-review — 종합 보안 체크리스트
- coding-standards — 보편적 코딩 표준
- frontend-patterns — React/Next.js 패턴
- frontend-slides — 뷰포트에 안전한 HTML 프레젠테이션 및 PPTX-to-web 변환
- article-writing — 노트 및 음성 참조를 바탕으로 하는 긴 형식의 글쓰기
- content-engine — 플랫폼 네이티브 소셜 콘텐츠 및 재가공
- market-research — 출처가 명시된 시장 및 경쟁사 조사
- investor-materials — 덱, 메모, 모델 및 원페이저(One-pagers)
- investor-outreach — 개인화된 투자자 연락 및 후속 조치
- backend-patterns — API 설계, 데이터베이스, 캐싱
- e2e-testing — Playwright E2E 테스트
- eval-harness — 평가(Eval) 중심 개발
- strategic-compact — 컨텍스트 관리
- api-design — REST API 설계 패턴
- verification-loop — 빌드, 테스트, 린트, 타입 체크, 보안

## MCP 서버

`~/.codex/config.toml`의 `[mcp_servers]` 섹션에 구성하십시오. GitHub, Context7, Memory 및 Sequential Thinking 서버에 대한 참조 구성은 `.codex/config.toml`을 확인하십시오.

## 멀티 에이전트 지원 (Multi-Agent Support)

Codex는 이제 실험적인 `features.multi_agent` 플래그를 통해 멀티 에이전트 워크플로우를 지원합니다.

- `.codex/config.toml`에서 `[features] multi_agent = true`로 활성화하십시오.
- `[agents.<name>]` 아래에 프로젝트 로컬 역할을 정의하십시오.
- `.codex/agents/` 아래의 TOML 레이어에 각 역할을 연결하십시오.
- Codex CLI 내에서 `/agent`를 사용하여 하위 에이전트를 검사하고 관리하십시오.

이 저장소의 샘플 역할 구성:
- `.codex/agents/explorer.toml` — 읽기 전용 정보 수집
- `.codex/agents/reviewer.toml` — 정확성/보안 리뷰
- `.codex/agents/docs-researcher.toml` — API 및 릴리스 노트 검증

## Claude Code와의 주요 차이점

| 기능 | Claude Code | Codex CLI |
|---------|------------|-----------|
| 후크 (Hooks) | 8개 이상의 이벤트 유형 | 아직 지원되지 않음 |
| 컨텍스트 파일 | CLAUDE.md + AGENTS.md | AGENTS.md만 사용 |
| 스킬 (Skills) | 플러그인을 통해 로드 | `.agents/skills/` 디렉토리 |
| 명령어 (Commands) | `/slash` 명령어 | 지침 기반 (Instruction-based) |
| 에이전트 | Subagent Task 도구 | `/agent` 및 `[agents.<name>]` 역할을 통한 멀티 에이전트 |
| 보안 | 후크 기반 강제 사항 | 지침 + 샌드박스 |
| MCP | 전체 지원 | 명령 기반(Command-based)만 지원 |

## 후크 없는 보안 (Security Without Hooks)

Codex에는 후크가 없으므로 보안 적용은 지침을 기반으로 합니다:
1. 항상 시스템 경계에서 입력값을 검증하십시오.
2. 비밀 정보를 절대 하드코딩하지 마십시오 — 환경 변수를 사용하십시오.
3. 커밋 전에 `npm audit` / `pip audit`을 실행하십시오.
4. 푸시하기 전에 매번 `git diff`를 리뷰하십시오.
5. 구성 파일에서 `sandbox_mode = "workspace-write"`를 사용하십시오.
