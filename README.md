**언어:** [English](README.md) | **한국어** | [简体中文](README.zh-CN.md) | [繁體中文](docs/zh-TW/README.md) | [日本語](docs/ja-JP/README.md)

# Everything Claude Code (모든 클로드 코드)

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![Forks](https://img.shields.io/github/forks/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/network/members)
[![Contributors](https://img.shields.io/github/contributors/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/graphs/contributors)
[![npm ecc-universal](https://img.shields.io/npm/dw/ecc-universal?label=ecc-universal%20weekly%20downloads&logo=npm)](https://www.npmjs.com/package/ecc-universal)
[![npm ecc-agentshield](https://img.shields.io/npm/dw/ecc-agentshield?label=ecc-agentshield%20weekly%20downloads&logo=npm)](https://www.npmjs.com/package/ecc-agentshield)
[![GitHub App Install](https://img.shields.io/badge/GitHub%20App-150%20installs-2ea44f?logo=github)](https://github.com/marketplace/ecc-tools)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> **50K+ 스타** | **6K+ 포크** | **30명의 기여자** | **5개 언어 지원** | **Anthropic 해커톤 우승작**

---

**AI 에이전트 하네스를 위한 성능 최적화 시스템. Anthropic 해커톤 우승자의 작품입니다.**

단순한 설정 모음이 아닙니다. 스킬, 본능, 메모리 최적화, 지속적 학습, 보안 스캔, 리서치 기반 개발을 아우르는 완체 시스템입니다. 실제 제품을 구축하며 10개월 이상 매일 집약적으로 사용하며 진화시킨 프로덕션급 에이전트, 후크, 명령어, 규칙 및 MCP 설정들이 포함되어 있습니다.

**Claude Code**, **Codex**, **Cowork** 및 기타 AI 에이전트 하네스에서 작동합니다.

---

## 가이드 (The Guides)

이 레포지토리는 원본 코드만 포함하고 있습니다. 가이드에서 모든 것을 설명합니다.

<table>
<tr>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" alt="Everything Claude Code 단축 가이드" />
</a>
</td>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" alt="Everything Claude Code 상세 가이드" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>단축 가이드(Shorthand)</b><br/>설정, 기초, 철학. <b>먼저 읽어보세요.</b></td>
<td align="center"><b>상세 가이드(Longform)</b><br/>토큰 최적화, 메모리 지속성, 평가(Evals), 병렬화.</td>
</tr>
</table>

| 주제                        | 학습 내용                                                        |
| --------------------------- | ---------------------------------------------------------------- |
| 토큰 최적화                 | 모델 선택, 시스템 프롬프트 슬리밍(Slimming), 백그라운드 프로세스 |
| 메모리 지속성               | 세션 간 컨텍스트를 자동으로 저장/로드하는 후크                   |
| 지속적 학습                 | 세션의 패턴을 재사용 가능한 스킬로 자동 추출                     |
| 검증 루프                   | 체크포인트 vs 지속적 평가, 채점자 유형, pass@k 메트릭            |
| 병렬화                      | Git worktrees, 캐스케이드(Cascade) 방식, 인스턴스 확장 시점      |
| 서브에이전트 오케스트레이션 | 컨텍스트 문제, 반복적 검색(Iterative retrieval) 패턴             |

---

## 새로운 소식 (What's New)

### v1.8.0 — 하네스 성능 시스템 (2026년 3월)

- **하네스 우선 릴리스** — ECC는 이제 단순한 설정팩이 아니라 AI 에이전트 하네스 성능 시스템으로 명확히 정의됩니다.
- **후크 안정성 개선** — SessionStart 루트 폴백, 중단 단계(Stop-phase) 세션 요약, 그리고 취약한 인라인 명령어를 대체하는 스크립트 기반 후크가 도입되었습니다.
- **후크 런타임 제어** — 후크 파일을 수정하지 않고도 `ECC_HOOK_PROFILE=minimal|standard|strict` 및 `ECC_DISABLED_HOOKS=...`를 통해 실행 시점을 제어할 수 있습니다.
- **새로운 하네스 명령어** — `/harness-audit`, `/loop-start`, `/loop-status`, `/quality-gate`, `/model-route`.
- **NanoClaw v2** — 모델 라우팅, 스킬 즉시 로드(Hot-load), 세션 브랜치/검색/내보내기/압축/메트릭 기능이 추가되었습니다.
- **크로스 하네스 호환성** — Claude Code, Cursor, OpenCode, Codex 앱/CLI 전반에서 동작이 일관되게 조정되었습니다.
- **997개 내부 테스트 통과** — 후크/런타임 리팩토링 및 호환성 업데이트 후 모든 테스트가 통과되었습니다.

### v1.7.0 — 크로스 플랫폼 확장 및 프레젠테이션 빌더 (2026년 2월)

- **Codex 앱 + CLI 지원** — `AGENTS.md` 기반의 직접적인 Codex 지원, 인스톨러 대상 지정 및 Codex 문서가 추가되었습니다.
- **`frontend-slides` 스킬** — 의존성 없는 HTML 프레젠테이션 빌더로, PPTX 변환 가이드 및 엄격한 뷰포트 맞춤 규칙이 포함되었습니다.
- **5개의 새로운 비즈니스/콘텐츠 스킬** — `article-writing`, `content-engine`, `market-research`, `investor-materials`, `investor-outreach`
- **광범위한 도구 지원** — 모든 주요 하네스에서 동일한 레포지토리가 깨끗하게 작동하도록 Cursor, Codex, OpenCode 지원을 강화했습니다.
- **992개 내부 테스트** — 플러그인, 후크, 스킬 및 패키징 전반에 걸쳐 검증 및 회귀 테스트 범위를 확대했습니다.

### v1.6.0 — Codex CLI, AgentShield 및 마켓플레이스 (2026년 2월)

- **Codex CLI 지원** — OpenAI Codex CLI 호환성을 위해 `codex.md`를 생성하는 새로운 `/codex-setup` 명령어가 추가되었습니다.
- **7개의 새로운 스킬** — `search-first`, `swift-actor-persistence`, `swift-protocol-di-testing`, `regex-vs-llm-structured-text`, `content-hash-cache-pattern`, `cost-aware-llm-pipeline`, `skill-stocktake`
- **AgentShield 통합** — Claude Code에서 직접 AgentShield를 실행하는 `/security-scan` 스킬이 추가되었습니다. (1282개 테스트, 102개 규칙)
- **GitHub 마켓플레이스** — [github.com/marketplace/ecc-tools](https://github.com/marketplace/ecc-tools)에서 Free/Pro/Enterprise 티어로 제공됩니다.
- **30개 이상의 커뮤니티 PR 머지** — 6개 언어에 걸쳐 30명의 기여자가 참여했습니다.
- **978개 내부 테스트** — 에이전트, 스킬, 명령어, 후크 및 규칙 전반에 걸쳐 검증 세트를 확대했습니다.

### v1.4.1 — 버그 수정 (2026년 2월)

- **인스팅트(Instinct) 임포트 시 내용 소실 문제 수정** — `/instinct-import` 중 프론트매터 이후의 내용(Action, Evidence, Examples 섹션)을 누락하던 `parse_instinct_file()` 함수가 수정되었습니다. (기여자 @ericcai0814)

### v1.4.0 — 다국어 규칙, 설치 마법사 및 PM2 (2026년 2월)

- **대화형 설치 마법사** — 머지/덮어쓰기 감지 기능이 포함된 가이드 설정 스킬인 `configure-ecc`가 추가되었습니다.
- **PM2 및 멀티 에이전트 오케스트레이션** — 복잡한 멀티 서비스 워크플로우 관리를 위한 6개의 새로운 명령어가 추가되었습니다. (`/pm2`, `/multi-plan` 등)
- **다국어 규칙 아키텍처** — 규칙 파일이 `common/` + 언어별 디렉토리 구조로 재구성되었습니다. 필요한 언어만 설치하십시오.
- **중국어(zh-CN) 번역** — 모든 에이전트, 명령어, 스킬 및 규칙(80개 이상의 파일)의 번역이 완료되었습니다.
- **GitHub 스폰서 지원** — 프로젝트를 후원할 수 있습니다.
- **개선된 CONTRIBUTING.md** — 각 기여 유형별 상세 PR 템플릿이 추가되었습니다.

### v1.3.0 — OpenCode 플러그인 지원 (2026년 2월)

- **완벽한 OpenCode 통합** — OpenCode의 플러그인 시스템을 통해 12개 에이전트, 24개 명령어, 16개 스킬이 후크를 지원합니다.
- **3개의 네이티브 커스텀 도구** — run-tests, check-coverage, security-audit
- **LLM용 문서** — 종합적인 OpenCode 문서를 담은 `llms.txt`

### v1.2.0 — 통합 명령어 및 스킬 (2026년 2월)

- **Python/Django 지원** — Django 패턴, 보안, TDD 및 검증 스킬
- **Java Spring Boot 스킬** — Spring Boot용 패턴, 보안, TDD 및 검증
- **세션 관리** — 세션 히스토리를 위한 `/sessions` 명령어
- **지속적 학습 v2** — 신뢰도 점수, 임포트/익스포트, 진화 기능이 포함된 인스팅트 기반 학습

전체 변경 이력은 [Releases](https://github.com/affaan-m/everything-claude-code/releases)에서 확인하십시오.

---

## 🚀 빠른 시작 (Quick Start)

2분 안에 시작하기:

### 1단계: 플러그인 설치

```bash
# 마켓플레이스 추가
/plugin marketplace add affaan-m/everything-claude-code

# 플러그인 설치
/plugin install everything-claude-code@everything-claude-code
```

### 2단계: 규칙 설치 (필수)

> ⚠️ **중요:** Claude Code 플러그인은 `rules`(규칙)를 자동으로 배포할 수 없습니다. 수동으로 설치하십시오:

```bash
# 먼저 레포지토리를 클론합니다
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# 권장: 인스톨러 사용 (공통 및 언어별 규칙을 안전하게 처리)
./install.sh typescript    # 또는 python, golang, swift, php 등
# 여러 언어 전달 가능:
# ./install.sh typescript python golang swift php
# 또는 대상(Target) 지정:
# ./install.sh --target cursor typescript
# ./install.sh --target antigravity typescript
```

수동 설치 지침은 `rules/` 폴더의 README를 참고하십시오.

### 3단계: 사용 시작

```bash
# 명령어 실행 (플러그인 설치 시 네임스페이스 포함 형태)
/everything-claude-code:plan "사용자 인증 추가"

# 수동 설치(옵션 2) 시에는 짧은 형식 사용 가능:
# /plan "사용자 인증 추가"

# 사용 가능한 명령어 확인
/plugin list everything-claude-code@everything-claude-code
```

✨ **끝났습니다!** 이제 16개 에이전트, 65개 스킬, 40개 명령어를 사용할 수 있습니다.

---

## 🌐 크로스 플랫폼 지원 (Cross-Platform Support)

이 플러그인은 **Windows, macOS, Linux**를 완벽하게 지원하며, 주요 IDE(Cursor, OpenCode, Antigravity) 및 CLI 하네스와 긴밀하게 통합됩니다. 모든 후크와 스크립트는 최대 호환성을 위해 Node.js로 작성되었습니다.

### 패키지 매니저 감지

플러그인은 다음 우선순위에 따라 선호하는 패키지 매니저(npm, pnpm, yarn, bun)를 자동으로 감지합니다:

1. **환경 변수**: `CLAUDE_PACKAGE_MANAGER`
2. **프로젝트 설정**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 필드
4. **락 파일(Lock file)**: package-lock.json, yarn.lock, pnpm-lock.yaml, bun.lockb 감지
5. **글로벌 설정**: `~/.claude/package-manager.json`
6. **폴백(Fallback)**: 사용 가능한 첫 번째 패키지 매니저

선호하는 패키지 매니저 설정 방법:

```bash
# 환경 변수 사용
export CLAUDE_PACKAGE_MANAGER=pnpm

# 글로벌 설정 사용
node scripts/setup-package-manager.js --global pnpm

# 프로젝트 설정 사용
node scripts/setup-package-manager.js --project bun

# 현재 설정 감지
node scripts/setup-package-manager.js --detect
```

또는 Claude Code에서 `/setup-pm` 명령어를 사용하십시오.

### 후크 런타임 제어

런타임 플래그를 사용하여 엄격도를 조정하거나 특정 후크를 일시적으로 비활성화할 수 있습니다:

```bash
# 후크 엄격도 프로필 (기본값: standard)
export ECC_HOOK_PROFILE=standard

# 비활성화할 후크 ID (쉼표로 구분)
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"
```

---

## 📦 구성 요소 (What's Inside)

이 레포지토리는 **Claude Code 플러그인**입니다. 직접 설치하거나 각 구성 요소를 수동으로 복사하여 사용할 수 있습니다.

```
everything-claude-code/
|-- .claude-plugin/   # 플러그인 및 마켓플레이스 매니페스트
|   |-- plugin.json         # 플러그인 메타데이터 및 컴포넌트 경로
|   |-- marketplace.json    # 마켓플레이스 카탈로그
|
|-- agents/           # 태스크 위임을 위한 전문 서브에이전트
|   |-- planner.md           # 기능 구현 계획
|   |-- architect.md         # 시스템 설계 결정
|   |-- tdd-guide.md         # 테스트 주도 개발
|   |-- code-reviewer.md     # 품질 및 보안 리뷰
|   |-- security-reviewer.md # 취약점 분석
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E 테스트
|   |-- refactor-cleaner.md  # 데드 코드 정리
|   |-- doc-updater.md       # 문서 동기화
|   |-- go-reviewer.md       # Go 코드 리뷰
|   |-- go-build-resolver.md # Go 빌드 에러 해결
|   |-- python-reviewer.md   # Python 코드 리뷰 (새로 추가)
|   |-- database-reviewer.md # 데이터베이스/Supabase 리뷰 (새로 추가)
|
|-- skills/           # 워크플로우 정의 및 도메인 지식
|   |-- coding-standards/           # 언어별 최선 관행
|   |-- clickhouse-io/              # ClickHouse 분석, 쿼리, 데이터 엔지니어링
|   |-- backend-patterns/           # API, 데이터베이스, 캐싱 패턴
|   |-- frontend-patterns/          # React, Next.js 패턴
|   |-- frontend-slides/            # HTML 슬라이드 데크 및 웹 프레젠테이션 워크플로우 (새로 추가)
|   |-- article-writing/            # AI 티가 나지 않는 고유한 어조의 장문 작성 (새로 추가)
|   |-- content-engine/             # 멀티 플랫폼 소셜 콘텐츠 제작 및 재가공 (새로 추가)
|   |-- market-research/            # 출처가 명확한 시장, 경쟁사, 투자자리서치 (새로 추가)
|   |-- investor-materials/         # 피치 데크, 요약서, 메모, 재무 모델 (새로 추가)
|   |-- investor-outreach/          # 개인화된 투자 유치 지원 및 후속 조치 (새로 추가)
|   |-- continuous-learning/        # 세션에서 패턴 자동 추출 (상세 가이드)
|   |-- continuous-learning-v2/     # 신뢰도 점수가 포함된 인스팅트 기반 학습
|   |-- iterative-retrieval/        # 서브에이전트를 위한 점진적 컨텍스트 정교화
|   |-- strategic-compact/          # 수동 압축 제안 (상세 가이드)
|   |-- tdd-workflow/               # TDD 방법론
|   |-- security-review/            # 보안 체크리스트
|   |-- eval-harness/               # 검증 루프 평가 (상세 가이드)
|   |-- verification-loop/          # 지속적 검증 (상세 가이드)
|   |-- videodb/                   # 비디오/오디오: 수집, 검색, 편집, 생성, 스트리밍 (새로 추가)
|   |-- golang-patterns/            # Go 관용구 및 최선 관행
|   |-- golang-testing/             # Go 테스트 패턴, TDD, 벤치마크
|   |-- cpp-coding-standards/         # C++ 코어 가이드라인 기반 표준 (새로 추가)
|   |-- cpp-testing/                # GoogleTest, CMake/CTest를 이용한 C++ 테스트 (새로 추가)
|   |-- django-patterns/            # Django 패턴, 모델, 뷰 (새로 추가)
|   |-- django-security/            # Django 보안 최선 관행 (새로 추가)
|   |-- django-tdd/                 # Django TDD 워크플로우 (새로 추가)
|   |-- django-verification/        # Django 검증 루프 (새로 추가)
|   |-- python-patterns/            # Python 관용구 및 최선 관행 (새로 추가)
|   |-- python-testing/             # pytest를 이용한 Python 테스트 (새로 추가)
|   |-- springboot-patterns/        # Java Spring Boot 패턴 (새로 추가)
|   |-- springboot-security/        # Spring Boot 보안 (새로 추가)
|   |-- springboot-tdd/             # Spring Boot TDD (새로 추가)
|   |-- springboot-verification/    # Spring Boot 검증 (새로 추가)
|   |-- configure-ecc/              # 대화형 설치 마법사 (새로 추가)
|   |-- security-scan/              # AgentShield 보안 오디터 통합 (새로 추가)
|   |-- java-coding-standards/     # Java 코딩 표준 (새로 추가)
|   |-- jpa-patterns/              # JPA/Hibernate 패턴 (새로 추가)
|   |-- postgres-patterns/         # PostgreSQL 최적화 패턴 (새로 추가)
|   |-- nutrient-document-processing/ # Nutrient API를 이용한 문서 처리 (새로 추가)
|   |-- project-guidelines-example/   # 프로젝트 전용 스킬 템플릿
|   |-- database-migrations/         # 마이그레이션 패턴 (Prisma, Drizzle, Django, Go) (새로 추가)
|   |-- api-design/                  # REST API 설계, 페이지네이션, 에러 응답 (새로 추가)
|   |-- deployment-patterns/         # CI/CD, Docker, 상태 확인, 롤백 (새로 추가)
|   |-- docker-patterns/            # Docker Compose, 네트워킹, 볼륨, 컨테이너 보안 (새로 추가)
|   |-- e2e-testing/                 # Playwright E2E 패턴 및 POM (새로 추가)
|   |-- content-hash-cache-pattern/  # 파일 처리를 위한 SHA-256 해시 캐싱 (새로 추가)
|   |-- cost-aware-llm-pipeline/     # LLM 비용 최적화, 모델 라우팅, 예산 추적 (새로 추가)
|   |-- regex-vs-llm-structured-text/ # 텍스트 파싱을 위한 정규식 vs LLM 결정 프레임워크 (새로 추가)
|   |-- swift-actor-persistence/     # 액터를 이용한 스레드 안전한 Swift 데이터 지속성 (새로 추가)
|   |-- swift-protocol-di-testing/   # 테스트 가능한 코드를 위한 프로토콜 기반 DI (새로 추가)
|   |-- search-first/               # 코딩 전 리서치 워크플로우 (새로 추가)
|   |-- skill-stocktake/            # 품질을 위한 스킬 및 명령어 점검 (새로 추가)
|   |-- liquid-glass-design/         # iOS 26 Liquid Glass 디자인 시스템 (새로 추가)
|   |-- foundation-models-on-device/ # Apple 온디바이스 모델 활용 (새로 추가)
|   |-- swift-concurrency-6-2/       # Swift 6.2 동시성 접근 (새로 추가)
|   |-- perl-patterns/             # 현대적인 Perl 5.36+ 관용구 (새로 추가)
|   |-- perl-security/             # Perl 보안 패턴, 테인트 모드, 안전한 I/O (새로 추가)
|   |-- perl-testing/              # Test2::V0를 이용한 Perl TDD (새로 추가)
|   |-- autonomous-loops/           # 자율 루프 패턴: 파이프라인, PR 루프, DAG (새로 추가)
|   |-- plankton-code-quality/      # Plankton 후크를 이용한 실시간 코드 품질 강제 (새로 추가)
|
|-- commands/         # 빠른 실행을 위한 슬래시 명령어
|   |-- tdd.md              # /tdd - 테스트 주도 개발
|   |-- plan.md             # /plan - 구현 계획 수립
|   |-- e2e.md              # /e2e - E2E 테스트 생성
|   |-- code-review.md      # /code-review - 품질 리뷰
|   |-- build-fix.md        # /build-fix - 빌드 에러 수정
|   |-- refactor-clean.md   # /refactor-clean - 데드 코드 제거
|   |-- learn.md            # /learn - 세션 도중 패턴 추출
|   |-- learn-eval.md       # /learn-eval - 패턴 추출, 평가 및 저장 (새로 추가)
|   |-- checkpoint.md       # /checkpoint - 검증 상태 저장
|   |-- verify.md           # /verify - 검증 루프 실행
|   |-- setup-pm.md         # /setup-pm - 패키지 매니저 설정
|   |-- go-review.md        # /go-review - Go 코드 리뷰 (새로 추가)
|   |-- go-test.md          # /go-test - Go TDD 워크플로우 (새로 추가)
|   |-- go-build.md         # /go-build - Go 빌드 에러 수정 (새로 추가)
|   |-- skill-create.md     # /skill-create - git 이력 기반 스킬 생성 (새로 추가)
|   |-- instinct-status.md  # /instinct-status - 학습된 인스팅트 확인 (새로 추가)
|   |-- instinct-import.md  # /instinct-import - 인스팅트 임포트 (새로 추가)
|   |-- instinct-export.md  # /instinct-export - 인스팅트 익스포트 (새로 추가)
|   |-- evolve.md           # /evolve - 인스팅트를 스킬로 클러스터링
|   |-- pm2.md              # /pm2 - PM2 서비스 생애주기 관리 (새로 추가)
|   |-- multi-plan.md       # /multi-plan - 멀티 에이전트 작업 분해 (새로 추가)
|   |-- multi-execute.md    # /multi-execute - 멀티 에이전트 워크플로우 (새로 추가)
|   |-- multi-backend.md    # /multi-backend - 백엔드 멀티 서비스 오케스트레이션 (새로 추가)
|   |-- multi-frontend.md   # /multi-frontend - 프론트엔드 멀티 서비스 오케스트레이션 (새로 추가)
|   |-- multi-workflow.md   # /multi-workflow - 일반 멀티 서비스 워크플로우 (새로 추가)
|   |-- orchestrate.md      # /orchestrate - 멀티 에이전트 조정
|   |-- sessions.md         # /sessions - 세션 히스토리 관리
|   |-- eval.md             # /eval - 기준에 따른 평가
|   |-- test-coverage.md    # /test-coverage - 테스트 커버리지 분석
|   |-- update-docs.md      # /update-docs - 문서 업데이트
|   |-- update-codemaps.md  # /update-codemaps - 코드맵 업데이트
|   |-- python-review.md    # /python-review - Python 코드 리뷰 (새로 추가)
|
|-- rules/            # 상시 준수 가이드라인 (~/.claude/rules/ 에 복사)
|   |-- README.md            # 구조 개요 및 설치 가이드
|   |-- common/              # 언어 무관 공통 원칭
|   |   |-- coding-style.md    # 불변성, 파일 구성
|   |   |-- git-workflow.md    # 커밋 형식, PR 프로세스
|   |   |-- testing.md         # TDD, 80% 커버리지 요구 사항
|   |   |-- performance.md     # 모델 선택, 컨텍스트 관리
|   |   |-- patterns.md        # 디자인 패턴, 스켈레톤 프로젝트
|   |   |-- hooks.md           # 후크 아키텍처, TodoWrite
|   |   |-- agents.md          # 서브에이전트 위임 시점
|   |   |-- security.md        # 필수 보안 검사
|   |-- typescript/          # TypeScript/JavaScript 전용
|   |-- python/              # Python 전용
|   |-- golang/              # Go 전용
|   |-- swift/               # Swift 전용
|   |-- php/                 # PHP 전용 (새로 추가)
```

|
|-- hooks/            # Trigger-based automations
|   |-- README.md                 # Hook documentation, recipes, and customization guide
|   |-- hooks.json                # All hooks config (PreToolUse, PostToolUse, Stop, etc.)
|   |-- memory-persistence/       # Session lifecycle hooks (Longform Guide)
|   |-- strategic-compact/        # Compaction suggestions (Longform Guide)
|
|-- scripts/          # Cross-platform Node.js scripts (NEW)
|   |-- lib/                     # Shared utilities
|   |   |-- utils.js             # Cross-platform file/path/system utilities
|   |   |-- package-manager.js   # Package manager detection and selection
|   |-- hooks/                   # Hook implementations
|   |   |-- session-start.js     # Load context on session start
|   |   |-- session-end.js       # Save state on session end
|   |   |-- pre-compact.js       # Pre-compaction state saving
|   |   |-- suggest-compact.js   # Strategic compaction suggestions
|   |   |-- evaluate-session.js  # Extract patterns from sessions
|   |-- setup-package-manager.js # Interactive PM setup
|
|-- tests/            # Test suite (NEW)
|   |-- lib/                     # Library tests
|   |-- hooks/                   # Hook tests
|   |-- run-all.js               # Run all tests
|
|-- contexts/         # Dynamic system prompt injection contexts (Longform Guide)
|   |-- dev.md              # Development mode context
|   |-- review.md           # Code review mode context
|   |-- research.md         # Research/exploration mode context
|
|-- examples/         # Example configurations and sessions
|   |-- CLAUDE.md             # Example project-level config
|   |-- user-CLAUDE.md        # Example user-level config
|   |-- saas-nextjs-CLAUDE.md   # Real-world SaaS (Next.js + Supabase + Stripe)
|   |-- go-microservice-CLAUDE.md # Real-world Go microservice (gRPC + PostgreSQL)
|   |-- django-api-CLAUDE.md      # Real-world Django REST API (DRF + Celery)
|   |-- rust-api-CLAUDE.md        # Real-world Rust API (Axum + SQLx + PostgreSQL) (NEW)
|
-

## 🛠️ 생태계 도구 (Ecosystem Tools)

### 스킬 생성기 (Skill Creator)

사용자의 레포지토리에서 Claude Code 스킬을 생성하는 두 가지 방법이 있습니다:

#### 옵션 A: 로컬 분석 (기본 제공)

외부 서비스 없이 로컬 분석을 위해 `/skill-create` 명령어를 사용하십시오:

```bash
/skill-create                    # 현재 레포지토리 분석
/skill-create --instincts        # 지속적 학습을 위한 인스팅트도 함께 생성
```

이 명령어는 로컬 git 이력을 분석하여 `SKILL.md` 파일을 생성합니다.

#### 옵션 B: GitHub 앱 (고급)

고급 기능(1만 개 이상의 커밋, 자동 PR, 팀 공유)이 필요한 경우:

[GitHub 앱 설치](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

```bash
# 이슈에 댓글 작성:
/skill-creator analyze

# 또는 기본 브랜치에 푸시할 때 자동 트리거
```

두 옵션 모두 다음을 생성합니다:

- **SKILL.md 파일** — Claude Code에서 즉시 사용 가능한 스킬
- **인스팅트 컬렉션** — 지속적 학습 v2를 위한 데이터
- **패턴 추출** — 커밋 이력으로부터 패턴 학습

### AgentShield — 보안 오디터

> Claude Code 해커톤(2026년 2월) 우승작. 1282개 테스트, 98% 커버리지, 102개 정적 분석 규칙 보유.

Claude Code 설정의 취약점, 오설정 및 인젝션 리스크를 스캔합니다.

```bash
# 빠른 스캔 (설치 불필요)
npx ecc-agentshield scan

# 안전한 이슈 자동 수정
npx ecc-agentshield scan --fix

# 세 명의 Opus 4.6 에이전트를 이용한 심층 분석
npx ecc-agentshield scan --opus --stream

# 처음부터 보안 설정 생성
npx ecc-agentshield init
```

**스캔 대상:** CLAUDE.md, settings.json, MCP 설정, 후크, 에이전트 정의 및 스킬을 5개 카테고리(시크릿 감지, 권한 오디팅, 후크 인젝션 분석, MCP 서버 리스크 프로파일링, 에이전트 설정 리뷰)에 걸쳐 검사합니다.

**`--opus` 플래그**는 공격자/방어자/감사자 파이프라인으로 구성된 세 명의 Claude Opus 4.6 에이전트를 실행합니다. 단순한 패턴 매칭이 아닌 적대적 추론(Adversarial reasoning)을 통해 리스크를 평가합니다.

[GitHub](https://github.com/affaan-m/agentshield) | [npm](https://www.npmjs.com/package/ecc-agentshield)

### 🔬 Plankton — 작성 시점 코드 품질 강제

Plankton(제작: @alxfazio)은 작성 시점의 코드 품질 강제를 위한 도구입니다. `PostToolUse` 후크를 통해 모든 파일 편집 시 포맷터와 20개 이상의 린터를 실행하며, 메인 에이전트가 놓친 부분을 수정하기 위해 Claude 서브프로세스를 가동합니다. 설정 보호 후크를 통해 에이전트가 코드를 고치는 대신 린터 설정을 수정하는 편법을 차단합니다. `skills/plankton-code-quality/`에서 자세한 가이드를 확인하십시오.

### 🧠 지속적 학습 v2 (Continuous Learning v2)

인스팅트 기반 학습 시스템이 사용자의 패턴을 자동으로 학습합니다:

```bash
/instinct-status        # 신뢰도와 함께 학습된 인스팅트 표시
/instinct-import <file> # 다른 사람의 인스팅트 임포트
/instinct-export        # 공유를 위해 인스팅트 익스포트
/evolve                 # 관련 인스팅트를 스킬로 클러스터링
```

자세한 문서는 `skills/continuous-learning-v2/`를 참조하십시오.

---

## 📋 요구 사항 (Requirements)

### Claude Code CLI 버전

**최소 버전: v2.1.0 이상**

이 플러그인은 후크 시스템 변경으로 인해 Claude Code CLI v2.1.0 이상이 필요합니다.

버전 확인:

```bash
claude --version
```

### 중요: 후크 자동 로드 동작

> ⚠️ **기여자 주의:** `.claude-plugin/plugin.json`에 `"hooks"` 필드를 추가하지 마십시오. 회귀 테스트에서 강제됩니다.

Claude Code v2.1+ 버전은 설치된 플러그인의 `hooks/hooks.json`을 관례에 따라 **자동으로 로드**합니다. 명시적으로 선언하면 중복 감지 에러가 발생합니다.

---

## 📥 설치 방법 (Installation)

### 옵션 1: 플러그인으로 설치 (권장)

가장 쉬운 방법입니다:

```bash
# 마켓플레이스로 추가
/plugin marketplace add affaan-m/everything-claude-code

# 플러그인 설치
/plugin install everything-claude-code@everything-claude-code
```

또는 `~/.claude/settings.json`에 직접 추가:

```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

> **참고:** Claude Code 플러그인 시스템은 현재 `rules` 배포를 지원하지 않습니다. 다음과 같이 수동으로 규칙을 설치해야 합니다:
>
> 1. 레포지토리 클론: `git clone https://github.com/affaan-m/everything-claude-code.git`
> 2. `rules/common/`의 내용을 `~/.claude/rules/common/`에 복사
> 3. 사용하는 스택에 맞는 언어별 규칙(예: `rules/typescript/`)을 `~/.claude/rules/typescript/`에 복사

---

### 🔧 옵션 2: 수동 설치

설치 대상을 직접 제어하고 싶은 경우:

```bash
# 레포지토리 클론
git clone https://github.com/affaan-m/everything-claude-code.git

# 에이전트를 Claude 설정으로 복사
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 규칙 복사 (공통 + 언어별)
cp -r everything-claude-code/rules/common/* ~/.claude/rules/common/
cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/typescript/

# 명령어 복사
cp everything-claude-code/commands/*.md ~/.claude/commands/

# 스킬 복사
cp -r everything-claude-code/.agents/skills/* ~/.claude/skills/
```

---

## 🎯 핵심 개념 (Key Concepts)

### 에이전트 (Agents)

제한된 범위 내에서 위임된 작업을 처리하는 서브에이전트입니다.

### 스킬 (Skills)

명령어나 에이전트에 의해 호출되는 워크플로우 정의입니다.

### 후크 (Hooks)

도구 이벤트 발생 시 실행되는 자동화입니다.

---

## 🗺️ 어떤 에이전트를 사용해야 하나요?

| 목표                   | 명령어                                      | 사용 에이전트        |
| ---------------------- | ------------------------------------------- | -------------------- |
| 신규 기능 계획         | `/everything-claude-code:plan "Add auth"` | planner              |
| 시스템 아키텍처 설계   | `/plan` + architect 에이전트              | architect            |
| 테스트 우선 코드 작성  | `/tdd`                                    | tdd-guide            |
| 작성한 코드 리뷰       | `/code-review`                            | code-reviewer        |
| 빌드 에러 수정         | `/build-fix`                              | build-error-resolver |
| 엔드투엔드 테스트 실행 | `/e2e`                                    | e2e-runner           |
| 보안 취약점 찾기       | `/security-scan`                          | security-reviewer    |
| 데드 코드 제거         | `/refactor-clean`                         | refactor-cleaner     |
| 문서 업데이트          | `/update-docs`                            | doc-updater          |

---

## ❓ 자주 묻는 질문 (FAQ)

<details>
<summary><b>Cursor / OpenCode / Codex / Antigravity에서 작동하나요?</b></summary>

예, ECC는 크로스 플랫폼을 지향합니다:

- **Cursor**: `.cursor/`에 사전 번역된 설정이 있습니다.
- **OpenCode**: `.opencode/`에서 플러그인을 지원합니다.
- **Codex**: macOS 앱과 CLI 모두 지원합니다.
- **Antigravity**: `.agent/`를 통해 워크플로우와 스킬이 긴밀하게 통합됩니다.
- **Claude Code**: 기본 타겟입니다.

</details>

---

## 🧪 테스트 실행 (Running Tests)

```bash
# 모든 테스트 실행
node tests/run-all.js

# 개별 테스트 실행
node tests/lib/utils.test.js
node tests/hooks/hooks.test.js
```

---

## 🤝 기여하기 (Contributing)

**여러분의 기여를 환영합니다.** 유용한 에이전트, 스킬, 후크 또는 개선된 규칙이 있다면 언제든 공유해 주세요! 자세한 가이드는 [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하십시오.

## Cursor IDE 지원 (Cursor IDE Support)

ECC는 Cursor의 네이티브 형식에 맞게 후크, 규칙, 에이전트, 스킬, 명령어 및 MCP 설정을 조정하여 **완벽한 Cursor IDE 지원**을 제공합니다.

### 빠른 시작 (Cursor)

```bash
# 사용 언어에 맞게 설치
./install.sh --target cursor typescript
./install.sh --target cursor python golang swift php
```

### 포함된 구성 요소

| 컴포넌트      | 개수      | 상세 내용                                                         |
| ------------- | --------- | ----------------------------------------------------------------- |
| 후크 이벤트   | 15개      | sessionStart, beforeShellExecution, afterFileEdit 등              |
| 후크 스크립트 | 16개      | 공유 어댑터를 통해 `scripts/hooks/`로 위임되는 Node.js 스크립트 |
| 규칙          | 34개      | 9개의 공통 규칙 + 25개의 언어별 규칙 (TS, Python, Go, Swift, PHP) |
| 에이전트      | 공유      | 루트의 AGENTS.md를 통해 Cursor가 네이티브로 읽음                  |
| 스킬          | 공유+번들 | 루트의 AGENTS.md 및 `.cursor/skills/`에 포함된 추가 스킬        |
| 명령어        | 공유      | 설치된 경우 `.cursor/commands/`에 포함                          |
| MCP 설정      | 공유      | 설치된 경우 `.cursor/mcp.json`에 포함                           |

---

## Codex macOS 앱 + CLI 지원

ECC는 macOS 앱과 CLI 모두에 대해 **퍼스트 클래스 Codex 지원**을 제공합니다.

### 빠른 시작 (Codex 앱 + CLI)

```bash
# 레포지토리에서 Codex CLI 실행 — AGENTS.md 및 .codex/를 자동 감지합니다.
codex

# 옵션: 글로벌 기본 설정을 홈 디렉토리에 복사
cp .codex/config.toml ~/.codex/config.toml
```

---

## 🔌 OpenCode 지원

ECC는 플러그인과 후크를 포함한 **완벽한 OpenCode 지원**을 제공합니다.

### 빠른 시작

```bash
# OpenCode 설치
npm install -g opencode

# 레포지토리 루트에서 실행
opencode
```

설정은 `.opencode/opencode.json`에서 자동으로 감지됩니다.

### 플랫폼별 기능 비교

| 기능                  | Claude Code | Cursor IDE       | Codex CLI         | OpenCode          |
| --------------------- | ----------- | ---------------- | ----------------- | ----------------- |
| **에이전트**    | 16개        | 공유 (AGENTS.md) | 공유 (AGENTS.md)  | 12개              |
| **명령어**      | 40개        | 공유             | 지침 기반         | 31개              |
| **스킬**        | 65개        | 공유             | 10개 (네이티브)   | 37개              |
| **후크 이벤트** | 8개 유형    | 15개 유형        | 미지원            | 11개 유형         |
| **규칙**        | 34개        | 34개             | 지침 기반         | 13개 지침         |
| **커스텀 도구** | 후크 방식   | 후크 방식        | 미지원            | 6개 네이티브 도구 |
| **MCP 서버**    | 14개        | 공유 (mcp.json)  | 4개 (명령어 기반) | 전체 지원         |

---

## 📖 배경 (Background)

저는 실험적 출시 단계부터 Claude Code를 사용해 왔습니다. 2025년 9월 Anthropic x Forum Ventures 해커톤에서 오직 Claude Code만을 사용하여 [zenith.chat](https://zenith.chat)을 구축해 우승했습니다. 이 설정들은 여러 프로덕션 애플리케이션을 통해 검증되었습니다.

---

## 토큰 최적화 (Token Optimization)

Claude Code 사용 비용은 토큰 소비를 어떻게 관리하느냐에 따라 달라집니다. 다음 설정은 품질을 유지하면서 비용을 크게 절감해 줍니다.

### 권장 설정

`~/.claude/settings.json`에 추가하십시오:

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  }
}
```

| 설정                                | 기본값 | 권장값           | 효과                                              |
| ----------------------------------- | ------ | ---------------- | ------------------------------------------------- |
| `model`                           | opus   | **sonnet** | 비용 약 60% 절감; 코딩 작업의 80% 이상 처리 가능  |
| `MAX_THINKING_TOKENS`             | 31,999 | **10,000** | 요청당 보이지 않는 'Thinking' 비용 약 70% 절감    |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 95     | **50**     | 더 일찍 압축 실행 — 긴 세션에서 품질 유지에 유리 |

심층적인 아키텍처 추론이 필요할 때만 Opus로 전환하십시오:

```
/model opus
```

### 전략적 압축 (Strategic Compaction)

이 플러그인에 포함된 `strategic-compact` 스킬은 95% 컨텍스트 임계치에 도달할 때까지 기다리는 대신, 논리적인 작업 지점에서 `/compact`를 실행하도록 제안합니다.

**압축 권장 시점:**

- 리서치 완료 후 구현 시작 전
- 마일스톤 달성 후 다음 단계 시작 전
- 디버깅 완료 후 기능 개발 재개 전
- 접근 방식이 실패하여 새로운 시도를 하기 전

---

## ⚠️ 중요 참고 사항

### 사용자 정의 (Customization)

이 설정들은 저의 워크플로우에 최적화되어 있습니다. 여러분은 다음을 고려해야 합니다:

1. 마음에 드는 부분부터 시작하십시오.
2. 자신의 기술 스택에 맞게 수정하십시오.
3. 사용하지 않는 기능은 제거하십시오.
4. 자신만의 패턴을 추가하십시오.

---

## 🔗 링크 (Links)

- **단축 가이드 (여기서 시작):** [Everything Claude Code 단축 가이드](https://x.com/affaanmustafa/status/2012378465664745795)
- **상세 가이드 (고급):** [Everything Claude Code 상세 가이드](https://x.com/affaanmustafa/status/2014040193557471352)
- **Follow:** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## 📄 라이선스 (License)

MIT - 자유롭게 사용하고, 필요에 따라 수정하며, 가능하다면 커뮤니티에 기여해 주세요.

---

**이 레포지토리가 도움이 되었다면 스타(Star)를 눌러주세요. 가이드를 읽고 멋진 제품을 만드시길 바랍니다.**
