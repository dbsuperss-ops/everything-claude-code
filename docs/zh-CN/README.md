**언어:** [English](../../README.md) | [繁體中文](../zh-TW/README.md) | [简体中文](README.md)

# Everything Claude Code (ECC)

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![Forks](https://img.shields.io/github/forks/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/network/members)
[![Contributors](https://img.shields.io/github/contributors/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/graphs/contributors)
[![npm ecc-universal](https://img.shields.io/npm/dw/ecc-universal?label=ecc-universal%20weekly%20downloads\&logo=npm)](https://www.npmjs.com/package/ecc-universal)
[![npm ecc-agentshield](https://img.shields.io/npm/dw/ecc-agentshield?label=ecc-agentshield%20weekly%20downloads\&logo=npm)](https://www.npmjs.com/package/ecc-agentshield)
[![GitHub App Install](https://img.shields.io/badge/GitHub%20App-150%20installs-2ea44f?logo=github)](https://github.com/marketplace/ecc-tools)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash\&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript\&logoColor=white)
![Python](https://img.shields.io/badge/-Python-3776AB?logo=python\&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go\&logoColor=white)
![Java](https://img.shields.io/badge/-Java-ED8B00?logo=openjdk\&logoColor=white)
![Markdown](https://img.shields.io/badge/-Markdown-000000?logo=markdown\&logoColor=white)

> **5만+ 별(stars)** | **6천+ 포크(forks)** | **30명의 기여자** | **6개 언어 지원** | **Anthropic 해커톤 우승작**

***

<div align="center">

**🌐 언어 / Language / 語言**

[**English**](../../README.md) | [简体中文](../../README.zh-CN.md) | [繁體中文](../zh-TW/README.md) | [日本語](../ja-JP/README.md)

</div>

***

**AI 에이전트 플랫폼을 위한 성능 최적화 시스템. Anthropic 해커톤 수상작.**

단순한 설정 모음이 아닙니다. 스킬, 본능, 메모리 최적화, 지속적 학습, 보안 스캔, 연구 우선 개발을 포함하는 완전한 시스템입니다. 10개월 이상의 집중적인 일상 사용과 실제 제품 구축 경험을 통해 진화한 생산 환경용 에이전트, 후크, 명령어, 규칙 및 MCP 구성을 제공합니다.

**Claude Code**, **Codex**, **Cowork** 및 기타 AI 에이전트 플랫폼에서 사용할 수 있습니다.

***

## 도입 및 배포

후원자, 플랫폼 또는 에코시스템 파트너에게 ECC를 소개할 때 다음 실시간 신호를 사용하십시오:

* **메인 패키지 설치 수:** npm의 [`ecc-universal`](https://www.npmjs.com/package/ecc-universal)
* **보안 도구 설치 수:** npm의 [`ecc-agentshield`](https://www.npmjs.com/package/ecc-agentshield)
* **GitHub 앱 배포:** [ECC 도구 마켓플레이스 목록](https://github.com/marketplace/ecc-tools)
* **자동 월간 지표 이슈:** `.github/workflows/monthly-metrics.yml` 기반
* **저장소 도입 신호:** 본 README 상단의 별(stars)/포크(forks)/기여자 배지

Claude Code 플러그인 설치 수는 현재 공용 API로 제공되지 않습니다. 파트너 보고 시에는 npm 지표, GitHub 앱 설치 수, 저장소 트래픽 및 브랜치 성장 지표를 결합하여 사용하십시오.

후원 미팅을 위한 지표 체크리스트와 명령어 스니펫은 [`docs/business/metrics-and-sponsorship.md`](../business/metrics-and-sponsorship.md)를 참조하십시오.

[**ECC 후원하기**](https://github.com/sponsors/affaan-m) | [후원 등급](SPONSORS.md) | [후원 계획](SPONSORING.md)

***

## 가이드

이 저장소에는 원본 코드만 포함되어 있습니다. 가이드에서 모든 것을 설명합니다.

<table>
<tr>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" alt="Everything Claude Code 요약 가이드" />
</a>
</td>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" alt="Everything Claude Code 상세 가이드" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>요약 가이드(Shorthand Guide)</b><br/>설정, 기초, 철학. <b>먼저 읽어보세요.</b></td>
<td align="center"><b>상세 가이드(Longform Guide)</b><br/>토큰 최적화, 메모리 유지, 평가, 병렬화.</td>
</tr>
</table>

| 주제 | 배울 내용 |
|-------|-------------------|
| 토큰 최적화 | 모델 선택, 시스템 프롬프트 간소화, 백그라운드 프로세스 |
| 메모리 유지 | 세션 간 컨텍스트를 자동 저장/로드하는 후크 |
| 지속적 학습 | 세션에서 패턴을 추출하여 재사용 가능한 스킬로 자동 변환 |
| 검증 루프 | 체크포인트 및 지속적 평가, 채점자 유형, pass@k 지표 |
| 병렬화 | Git 작업 트리(Working Tree), 계층적 접근 방식, 인스턴스 확장 시기 |
| 서브 에이전트 오케스트레이션 | 컨텍스트 문제, 반복적 검색 패턴 |

***

## 최신 소식

### v1.8.0 — 플랫폼 성능 시스템 (2026년 3월)

* **플랫폼 우선 릴리스** — ECC는 이제 단순한 설정 패키지가 아닌, 에이전트 플랫폼 성능 시스템으로 명확하게 설계되었습니다.
* **후크 신뢰성 대개편** — SessionStart 루트 폴백, Stop 단계 세션 요약, 취약한 인라인 후크를 스크립트 기반 후크로 대체.
* **후크 런타임 제어** — `ECC_HOOK_PROFILE=minimal|standard|strict` 및 `ECC_DISABLED_HOOKS=...`를 통해 파일 수정 없이 런타임 제어 가능.
* **새로운 플랫폼 명령어** — `/harness-audit`, `/loop-start`, `/loop-status`, `/quality-gate`, `/model-route`.
* **NanoClaw v2** — 모델 라우팅, 스킬 핫 로드, 세션 분기/검색/내보내기/압축/지표 추가.
* **플랫폼 간 일관성** — Claude Code, Cursor, OpenCode, Codex 앱/CLI에서 더욱 통일된 동작 제공.
* **997개 내부 테스트 통과** — 후크/런타임 리팩토링 및 호환성 업데이트 후 전체 테스트 슈트 통과.

### v1.7.0 — 플랫폼 확장 및 프레젠테이션 생성기 (2026년 2월)

* **Codex 앱 + CLI 지원** — `AGENTS.md` 기반의 직접적인 Codex 지원, 설치 타겟 지정 및 Codex 문서 추가.
* **`frontend-slides` 스킬** — 의존성 없는 HTML 프레젠테이션 생성기, PPTX 변환 가이드 및 엄격한 뷰포트 적응 규칙 포함.
* **5개의 새로운 일반 비즈니스/콘텐츠 스킬** — `article-writing`, `content-engine`, `market-research`, `investor-materials`, `investor-outreach`.
* **더 넓은 도구 지원** — Cursor, Codex, OpenCode 지원 강화로 동일한 코드베이스를 모든 주요 플랫폼에 깔끔하게 배포 가능.
* **992개 내부 테스트** — 플러그인, 후크, 스킬 및 패키징 분야에서 검증 및 회귀 테스트 커버리지 확장.

### v1.6.0 — Codex CLI, AgentShield 및 마켓플레이스 (2026년 2월)

* **Codex CLI 지원** — 새로운 `/codex-setup` 명령어로 OpenAI Codex CLI 호환성을 위한 `codex.md` 생성.
* **7개의 신규 스킬** — `search-first`, `swift-actor-persistence`, `swift-protocol-di-testing`, `regex-vs-llm-structured-text`, `content-hash-cache-pattern`, `cost-aware-llm-pipeline`, `skill-stocktake`.
* **AgentShield 통합** — `/security-scan` 스킬로 Claude Code에서 직접 AgentShield 실행; 1,282개 테스트, 102개 규칙 제공.
* **GitHub 마켓플레이스** — ECC 도구 GitHub 앱이 [github.com/marketplace/ecc-tools](https://github.com/marketplace/ecc-tools)에서 제공됨 (무료/프로/엔터프라이즈).
* **30개 이상의 커뮤니티 PR 병합** — 6개 언어, 30명의 기여자 참여.
* **978개 내부 테스트** — 에이전트, 스킬, 명령어, 후크 및 규칙 분야의 검증 슈트 확장.

### v1.4.1 — 버그 수정 (2026년 2월)

* **본능(Instinct) 가져오기 시 내용 유실 문제 해결** — `parse_instinct_file()`이 `/instinct-import` 중에 frontmatter 이후의 내용(Action, Evidence, Examples 섹션)을 자동으로 삭제하던 문제 해결. 커뮤니티 기여자 @ericcai0814 덕분에 수정됨 ([#148](https://github.com/affaan-m/everything-claude-code/issues/148), [#161](https://github.com/affaan-m/everything-claude-code/pull/161)).

### v1.4.0 — 다중 언어 규칙, 설치 마법사 및 PM2 (2026년 2월)

* **대화형 설치 마법사** — 새로운 `configure-ecc` 스킬이 병합/덮어쓰기 감지를 포함한 가이드 설정을 제공합니다.
* **PM2 및 다중 에이전트 오케스트레이션** — 복잡한 다중 서비스 워크플로우 관리를 위한 6개의 새로운 명령어 추가 (`/pm2`, `/multi-plan`, `/multi-execute`, `/multi-backend`, `/multi-frontend`, `/multi-workflow`).
* **다중 언어 규칙 아키텍처** — 규칙 파일을 `common/` + `typescript/` + `python/` + `golang/` 디렉토리로 재구성하여 필요한 언어만 설치 가능.
* **중국어(简体中文) 번역** — 모든 에이전트, 명령어, 스킬 및 규칙에 대한 완전한 번역 (80개 이상의 파일).
* **GitHub Sponsors 지원** — 프로젝트 후원 가능.
* **CONTRIBUTING.md 강화** — 기여 유형별 상세 PR 템플릿 제공.

### v1.3.0 — OpenCode 플러그인 지원 (2026년 2월)

* **완전한 OpenCode 통합** — 12개의 에이전트, 24개의 명령어, 16개의 스킬, OpenCode 플러그인 시스템을 통한 후크 지원 (20개 이상의 이벤트 유형).
* **3개의 네이티브 커스텀 도구** — run-tests, check-coverage, security-audit.
* **LLM 문서** — 포괄적인 OpenCode 문서를 위한 `llms.txt` 제공.

### v1.2.0 — 통합 명령어 및 스킬 (2026년 2월)

* **Python/Django 지원** — Django 패턴, 보안, TDD 및 검증 스킬 추가.
* **Java Spring Boot 스킬** — Spring Boot 패턴, 보안, TDD 및 검증 추가.
* **세션 관리** — 세션 기록을 확인하기 위한 `/sessions` 명령어 추가.
* **지속적 학습 v2** — 신뢰도 점수, 가져오기/내보내기, 진화 기능을 포함한 본능 기반 학습.

전체 업데이트 기록은 [Releases](https://github.com/affaan-m/everything-claude-code/releases)를 참조하십시오.

***

## 🚀 빠른 시작

2분 안에 시작하기:

### 1단계: 플러그인 설치

```bash
# 마켓플레이스 추가
/plugin marketplace add affaan-m/everything-claude-code

# 플러그인 설치
/plugin install everything-claude-code@everything-claude-code
```

### 2단계: 규칙 설치 (필수)

> ⚠️ **중요:** Claude Code 플러그인은 `rules`를 자동으로 배포할 수 없습니다. 수동으로 설치하십시오:

```bash
# 먼저 저장소 클론
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# 권장: 설치 스크립트 사용 (공통 및 언어 규칙을 안전하게 처리)
./install.sh typescript    # 또는 python 또는 golang
# 여러 언어 설치 시:
# ./install.sh typescript python golang
# Cursor 대상:
# ./install.sh --target cursor typescript
# Antigravity 대상:
# ./install.sh --target antigravity typescript
```

수동 설치 지침은 `rules/` 폴더의 README를 참조하십시오.

### 3단계: 시작하기

```bash
# 명령어 시도 (플러그인 설치 시 네임스페이스 형식 사용)
/everything-claude-code:plan "사용자 인증 기능 추가"

# 수동 설치(옵션 2) 시에는 짧은 형식 사용 가능:
# /plan "사용자 인증 기능 추가"

# 사용 가능한 명령어 확인
/plugin list everything-claude-code@everything-claude-code
```

✨ **완료!** 이제 16개의 에이전트, 65개의 스킬, 40개의 명령어를 사용할 수 있습니다.

***

## 🌐 크로스 플랫폼 지원

본 플러그인은 **Windows, macOS, Linux**를 완벽하게 지원하며 주요 IDE(Cursor, OpenCode, Antigravity) 및 CLI 플랫폼과 긴밀하게 통합됩니다. 모든 후크와 스크립트는 최대 호환성을 위해 Node.js로 재작성되었습니다.

### 패키지 관리자 감지

플러그인은 다음과 같은 우선순위로 선호하는 패키지 관리자(npm, pnpm, yarn 또는 bun)를 자동 감지합니다:

1. **환경 변수**: `CLAUDE_PACKAGE_MANAGER`
2. **프로젝트 설정**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 필드
4. **잠금 파일**: package-lock.json, yarn.lock, pnpm-lock.yaml 또는 bun.lockb 감지
5. **전역 설정**: `~/.claude/package-manager.json`
6. **폴백**: 사용 가능한 첫 번째 패키지 관리자

선호하는 패키지 관리자 설정 방법:

```bash
# 환경 변수 사용
export CLAUDE_PACKAGE_MANAGER=pnpm

# 전역 설정 사용
node scripts/setup-package-manager.js --global pnpm

# 프로젝트 설정 사용
node scripts/setup-package-manager.js --project bun

# 현재 설정 감지
node scripts/setup-package-manager.js --detect
```

또는 Claude Code에서 `/setup-pm` 명령어를 사용할 수도 있습니다.

### 후크 런타임 제어

런타임 플래그를 사용하여 엄격도를 조정하거나 특정 후크를 일시적으로 비활성화할 수 있습니다:

```bash
# 후크 엄격도 프로필 (기본값: standard)
export ECC_HOOK_PROFILE=standard

# 비활성화할 후크 ID (쉼표로 구분)
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"
```

***

## 📦 포함 내용

이 저장소는 **Claude Code 플러그인**입니다. 직접 설치하거나 구성 요소를 수동으로 복사할 수 있습니다.

```
everything-claude-code/
|-- .claude-plugin/   # 플러그인 및 마켓플레이스 구성
|   |-- plugin.json         # 플러그인 메타데이터 및 컴포넌트 경로
|   |-- marketplace.json    # 마켓플레이스 디렉토리
|
|-- agents/           # 특정 작업을 위한 전용 서브 에이전트
|   |-- planner.md           # 구현 계획 수립
|   |-- architect.md         # 시스템 설계 결정
|   |-- tdd-guide.md         # 테스트 주도 개발
|   |-- code-reviewer.md     # 품질 및 보안 리뷰
|   |-- security-reviewer.md # 취약점 분석
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2E 테스트
|   |-- refactor-cleaner.md  # 데드 코드 정리
|   |-- doc-updater.md       # 문서 동기화
|   |-- go-reviewer.md       # Go 코드 리뷰
|   |-- go-build-resolver.md # Go 빌드 에러 수정
|   |-- python-reviewer.md   # Python 코드 리뷰 (신규)
|   |-- database-reviewer.md # 데이터베이스 / Supabase 리뷰 (신규)
|
|-- skills/           # 워크플로우 정의 및 도메인 지식
|   |-- coding-standards/           # 언어별 최선 관행
|   |-- clickhouse-io/              # ClickHouse 분석 및 데이터 엔지니어링
|   |-- backend-patterns/           # API, DB, 캐시 패턴
|   |-- frontend-patterns/          # React, Next.js 패턴
|   |-- frontend-slides/            # HTML 슬라이드 생성 및 웹 프레젠테이션 (신규)
|   |-- article-writing/            # AI 톤을 배제한 지정 스타일 장문 작성 (신규)
|   |-- content-engine/             # 다중 플랫폼 콘텐츠 생성 워크플로우 (신규)
|   |-- market-research/            # 출처가 명시된 시장 및 경쟁사 연구 (신규)
|   |-- investor-materials/         # IR 피치덱, 재무 모델링 (신규)
|   |-- investor-outreach/          # 개인화된 투자 외랜 및 후속 조치 (신규)
|   |-- continuous-learning/        # 세션 패턴 자동 추출 (Longform Guide)
|   |-- continuous-learning-v2/     # 본능 기반 학습 및 신뢰도 점수
|   |-- iterative-retrieval/        # 서브 에이전트의 점진적 컨텍스트 최적화
|   |-- strategic-compact/          # 수동 압축 제안 (Longform Guide)
|   |-- tdd-workflow/               # TDD 방법론
|   |-- security-review/            # 보안 체크리스트
|   |-- eval-harness/               # 검증 루프 평가 (Longform Guide)
|   |-- verification-loop/          # 지속적 검증 (Longform Guide)
|   |-- golang-patterns/            # Go 관용구 및 최선 관행
|   |-- golang-testing/             # Go 테스트 패턴, TDD, 벤치마크
|   |-- cpp-coding-standards/       # C++ Core Guidelines 준수 (신규)
|   |-- cpp-testing/                # GoogleTest, CMake 기반 C++ 테스트 (신규)
|   |-- django-patterns/            # Django 패턴, 모델 및 뷰 (신규)
|   |-- django-security/            # Django 보안 최선 관행 (신규)
|   |-- django-tdd/                 # Django TDD 워크플로우 (신규)
|   |-- django-verification/        # Django 검증 루프 (신규)
|   |-- python-patterns/            # Python 관용구 및 최선 관행 (신규)
|   |-- python-testing/             # pytest 기반 Python 테스트 (신규)
|   |-- springboot-patterns/        # Java Spring Boot 패턴 (신규)
|   |-- springboot-security/        # Spring Boot 보안 (신규)
|   |-- springboot-tdd/             # Spring Boot TDD (신규)
|   |-- springboot-verification/    # Spring Boot 검증 프로세스 (신규)
|   |-- configure-ecc/              # 대화형 설치 마법사 (신규)
|   |-- security-scan/              # AgentShield 보안 감사 통합 (신규)
|   |-- java-coding-standards/     # Java 코딩 표준 (신규)
|   |-- jpa-patterns/              # JPA/Hibernate 패턴 (신규)
|   |-- postgres-patterns/         # PostgreSQL 최적화 패턴 (신규)
|   |-- nutrient-document-processing/ # Nutrient API를 이용한 문서 처리 (신규)
|   |-- project-guidelines-example/   # 프로젝트 전용 스킬 템플릿
|   |-- database-migrations/         # DB 마이그레이션 패턴 (Prisma, Django 등) (신규)
|   |-- api-design/                  # REST API 설계 및 오류 응답 (신규)
|   |-- deployment-patterns/         # CI/CD, Docker, 상태 확인 및 롤백 (신규)
|   |-- docker-patterns/            # Docker Compose, 네트워크, 컨테이너 보안 (신규)
|   |-- e2e-testing/                 # Playwright E2E 패턴 (신규)
|   |-- content-hash-cache-pattern/  # SHA-256 해시 기반 파일 처리 캐싱 (신규)
|   |-- cost-aware-llm-pipeline/     # LLM 비용 최적화 및 모델 라우팅 (신규)
|   |-- regex-vs-llm-structured-text/ # 텍스트 파싱 결정 프레임워크 (신규)
|   |-- swift-actor-persistence/     # Actor 기반 스레드 안전 데이터 유지 (신규)
|   |-- swift-protocol-di-testing/   # 프로토콜 기반 의존성 주입 테스트 (신규)
|   |-- search-first/               # 연구 우선 코딩 워크플로우 (신규)
|   |-- skill-stocktake/            # 스킬 및 명령어 품질 감사 (신규)
|   |-- liquid-glass-design/         # iOS 26 Liquid Glass 디자인 시스템 (신규)
|   |-- foundation-models-on-device/ # Apple 온디바이스 LLM (신규)
|   |-- swift-concurrency-6-2/       # Swift 6.2 동시성 모델 (신규)
|   |-- autonomous-loops/           # 자동화 루프 패턴 (신규)
|   |-- plankton-code-quality/      # Plankton 후크 기반 코드 품질 검사 (신규)
|
|-- commands/         # 빠른 실행을 위한 슬래시 명령어
|   |-- tdd.md              # /tdd - 테스트 주도 개발
|   |-- plan.md             # /plan - 구현 계획
|   |-- e2e.md              # /e2e - E2E 테스트 생성
|   |-- code-review.md      # /code-review - 코드 품질 리뷰
|   |-- build-fix.md        # /build-fix - 빌드 오류 수정
|   |-- refactor-clean.md   # /refactor-clean - 데드 코드 삭제
|   |-- learn.md            # /learn - 패턴 추출 (Longform Guide)
|   |-- learn-eval.md       # /learn-eval - 패턴 추출 및 평가 (신규)
|   |-- checkpoint.md       # /checkpoint - 검증 상태 저장 (Longform Guide)
|   |-- verify.md           # /verify - 검증 루프 실행 (Longform Guide)
|   |-- setup-pm.md         # /setup-pm - 패키지 관리자 설정
|   |-- go-review.md        # /go-review - Go 코드 리뷰 (신규)
|   |-- go-test.md          # /go-test - Go TDD 워크플로우 (신규)
|   |-- go-build.md         # /go-build - Go 빌드 에러 수정 (신규)
|   |-- skill-create.md     # /skill-create - git 이력 기반 스킬 생성 (신규)
|   |-- instinct-status.md  # /instinct-status - 학습된 본능 확인 (신규)
|   |-- instinct-import.md  # /instinct-import - 본능 규칙 가져오기 (신규)
|   |-- instinct-export.md  # /instinct-export - 본능 규칙 내보내기 (신규)
|   |-- evolve.md           # /evolve - 본능을 스킬로 클러스터링
|   |-- pm2.md              # /pm2 - PM2 서비스 관리 (신규)
|   |-- multi-plan.md       # /multi-plan - 다중 에이전트 작업 분해 (신규)
|   |-- multi-execute.md    # /multi-execute - 오케스트레이션 워크플로우 (신규)
|   |-- multi-backend.md    # /multi-backend - 백엔드 다중 서비스 오케스트레이션 (신규)
|   |-- multi-frontend.md   # /multi-frontend - 프론트엔드 다중 서비스 오케스트레이션 (신규)
|   |-- multi-workflow.md   # /multi-workflow - 범용 서비스 워크플로우 (신규)
|   |-- orchestrate.md      # /orchestrate - 에이전트 협업
|   |-- sessions.md         # /sessions - 세션 기록 관리
|   |-- eval.md             # /eval - 기준 기반 평가
|   |-- test-coverage.md    # /test-coverage - 테스트 커버리지 분석
|   |-- update-docs.md      # /update-docs - 문서 업데이트
|   |-- update-codemaps.md  # /update-codemaps - 코드 지도 업데이트
|   |-- python-review.md    # /python-review - Python 코드 리뷰 (신규)
|
|-- rules/            # 항상 준수해야 할 규칙
|   |-- README.md            # 구조 개요 및 설치 가이드
|   |-- common/              # 언어 공통 원칙
|   |   |-- coding-style.md    # 불변성, 파일 구성
|   |   |-- git-workflow.md    # 커밋 형식, PR 프로세스
|   |   |-- testing.md         # TDD, 80% 커버리지 요구 사항
|   |   |-- performance.md     # 모델 선택, 컨텍스트 관리
|   |   |-- patterns.md        # 디자인 패턴, 스켈레톤 프로젝트
|   |   |-- hooks.md           # 후크 아키텍처, TodoWrite
|   |   |-- agents.md          # 서브 에이전트 위임 시점
|   |   |-- security.md        # 필수 보안 점검
|   |-- typescript/          # TS/JS 전용
|   |-- python/              # Python 전용
|   |-- golang/              # Go 전용
|
|-- hooks/            # 트리거 기반 자동화
|   |-- README.md                 # 후크 문서 및 사용자 정의 가이드
|   |-- hooks.json                # 전체 후크 구성
|   |-- memory-persistence/       # 세션 생명주기 후크 (Longform Guide)
|   |-- strategic-compact/        # 압축 제안 (Longform Guide)
|
|-- scripts/          # 크로스 플랫폼 Node.js 스크립트 (신규)
|   |-- lib/                     # 공유 라이브러리
|   |   |-- utils.js             # 파일/경로 유틸리티
|   |   |-- package-manager.js   # 패키지 관리자 감지
|   |-- hooks/                   # 후크 구현체
|   |   |-- session-start.js     # 세션 시작 시 컨텍스트 로드
|   |   |-- session-end.js       # 세션 종료 시 상태 저장
|   |   |-- pre-compact.js       # 압축 전 상태 저장
|   |   |-- suggest-compact.js   # 전략적 압축 제안
|   |   |-- evaluate-session.js  # 패턴 추출
|   |-- setup-package-manager.js # 패키지 관리자 설정 도구
|
|-- tests/            # 테스트 슈트 (신규)
|   |-- lib/                     # 라이브러리 테스트
|   |-- hooks/                   # 후크 테스트
|   |-- run-all.js               # 전체 테스트 실행
|
|-- contexts/         # 동적 시스템 프롬프트 주입 (Longform Guide)
|   |-- dev.md              # 개발 모드 컨텍스트
|   |-- review.md           # 리뷰 모드 컨텍스트
|   |-- research.md         # 연구 모드 컨텍스트
|
|-- examples/         # 예시 구성 및 세션
|   |-- CLAUDE.md             # 프로젝트 설정 예시
|   |-- user-CLAUDE.md        # 사용자 설정 예시
|   |-- saas-nextjs-CLAUDE.md   # SaaS 예시 (Next.js + Supabase)
|   |-- go-microservice-CLAUDE.md # Go 서비스 예시 (gRPC + PostgreSQL)
|   |-- django-api-CLAUDE.md      # Django REST API 예시 (DRF + Celery)
|   |-- rust-api-CLAUDE.md        # Rust API 예시 (Axum + SQLx) (신규)
|
|-- mcp-configs/      # MCP 서버 구성
|   |-- mcp-servers.json    # GitHub, Supabase, Vercel 등
|
|-- marketplace.json  # 자체 호스팅 마켓플레이스 구성
```

***

## 🛠️ 생태계 도구

### 스킬 생성기 (Skill Creator)

저장소에서 Claude Code 스킬을 생성하는 두 가지 방법:

#### 옵션 A: 로컬 분석 (내장)

외부 서비스 없이 `/skill-create` 명령어로 로컬 분석을 수행하십시오:

```bash
/skill-create                    # 현재 저장소 분석
/skill-create --instincts        # 지속적 학습을 위한 본능 규칙도 생성
```

이 명령어는 로컬 git 이력을 분석하고 `SKILL.md` 파일을 생성합니다.

#### 옵션 B: GitHub 앱 (고급)

1만 개 이상의 커밋, 자동 PR, 팀 공유 등 고급 기능이 필요한 경우:

[GitHub 앱 설치](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

```bash
# 이슈에 댓글 작성:
/skill-creator analyze

# 또는 기본 브랜치 푸시 시 자동 트리거
```

두 옵션 모두 다음을 생성합니다:
* **SKILL.md 파일** - 즉시 사용 가능한 스킬 정의
* **본능(Instinct) 컬렉션** - 지속적 학습 v2용
* **패턴 추출** - 커밋 이력 기반 학습

### AgentShield — 보안 감사 도구

> Claude Code 해커톤(2026년 2월) 수상작. 1,282개 테스트, 98% 커버리지, 102개의 정적 분석 규칙 제공.

Claude Code 구성을 스캔하여 취약점, 잘못된 설정 및 주입 리스크를 찾아냅니다.

```bash
# 빠른 스캔 (설치 불필요)
npx ecc-agentshield scan

# 안전한 문제 자동 수정
npx ecc-agentshield scan --fix

# 세 개의 Opus 4.6 에이전트를 이용한 심층 분석
npx ecc-agentshield scan --opus --stream

# 새로운 보안 구성 생성
npx ecc-agentshield init
```

**스캔 대상:** CLAUDE.md, settings.json, MCP 구성, 후크, 에이전트 정의 및 5개 카테고리의 스킬 — 비밀 정보 감지(14개 패턴), 권한 감사, 후크 주입 분석, MCP 서버 리스크 프로파일링, 에이전트 구성 리뷰.

**`--opus` 플래그**는 레드팀/블루팀/감사자 파이프라인에서 세 개의 Claude Opus 4.6 에이전트를 실행합니다. 단순 패턴 매칭이 아닌 대립적 추론을 수행합니다.

**출력 형식:** 터미널(A-F 등급), JSON(CI 파이프라인용), Markdown, HTML.

Claude Code에서 `/security-scan`을 사용하여 실행하거나 CI에 추가하십시오.

[GitHub](https://github.com/affaan-m/agentshield) | [npm](https://www.npmjs.com/package/ecc-agentshield)

### 🔬 Plankton — 작성 시 코드 품질 실시간 적용

Plankton은 작성 시 코드 품질을 강제하기 위한 권장 도구입니다. PostToolUse 후크를 통해 매 파일 편집 시 20개 이상의 코드 검사기를 실행하고 에이전트가 놓친 부분을 해결합니다. 자세한 내용은 `skills/plankton-code-quality/`를 참조하십시오.

### 🧠 지속적 학습 v2 (Continuous Learning v2)

본능(Instinct) 기반 학습 시스템이 사용자의 패턴을 자동으로 학습합니다:

```bash
/instinct-status        # 신뢰도 기반의 학습된 본능 확인
/instinct-import <file> # 외부 본능 규칙 가져오기
/instinct-export        # 본능 규칙 공유를 위해 내보내기
/evolve                 # 관련 본능을 스킬로 클러스터링
```

상세 문서는 `skills/continuous-learning-v2/`를 참조하십시오.

***

## 📋 요구 사항

### Claude Code CLI 버전

**최소 버전: v2.1.0 이상**

플러그인 시스템의 후크 처리 방식이 변경되었으므로 v2.1.0+ 버전이 필요합니다.

버전 확인:
```bash
claude --version
```

### 중요: 후크 자동 로드 동작

> ⚠️ **기여자 주의:** `.claude-plugin/plugin.json`에 `"hooks"` 필드를 추가하지 마십시오. 회귀 테스트에 의해 강제됩니다.

Claude Code v2.1+는 설치된 플러그인의 `hooks/hooks.json`을 자동으로 로드합니다. 명시적으로 선언하면 중복 로드 오류가 발생합니다.

***

## 📥 설치 방법

### 옵션 1: 플러그인으로 설치 (권장)

가장 간단한 방법입니다:

```bash
# 마켓플레이스 추가
/plugin marketplace add affaan-m/everything-claude-code

# 플러그인 설치
/plugin install everything-claude-code@everything-claude-code
```

또는 `~/.claude/settings.json`에 직접 추가할 수도 있습니다.

> **주의:** 플러그인 시스템은 `rules` 배포를 지원하지 않습니다. 반드시 수동으로 규칙을 설치하십시오:
> 
> ```bash
> # 저장소 클론 후 규칙 복사
> cp -r everything-claude-code/rules/common/* ~/.claude/rules/
> cp -r everything-claude-code/rules/typescript/* ~/.claude/rules/ # 기술 스택에 맞춤
> ```

***

### 🔧 옵션 2: 수동 설치

설치 내용을 직접 제어하고 싶은 경우:

```bash
# 저장소 클론
git clone https://github.com/affaan-m/everything-claude-code.git

# 에이전트 복사
cp everything-claude-code/agents/*.md ~/.claude/agents/

# 규칙 복사 (공통 및 언어별)
cp -r everything-claude-code/rules/common/* ~/.claude/rules/
# 필요한 언어 규칙 추가
```

#### 후크를 settings.json에 추가

`hooks/hooks.json`의 내용을 `~/.claude/settings.json`으로 복사하십시오.

#### MCP 구성

`mcp-configs/mcp-servers.json`의 내용을 `~/.claude.json`으로 복사하고 API 키를 본인의 값으로 바꾸십시오.

***

## 🎯 핵심 개념

### 에이전트 (Agents)

서브 에이전트는 제한된 범위의 작업을 처리합니다. 예: `code-reviewer`는 코드 품질 및 보안을 담당합니다.

### 스킬 (Skills)

스킬은 명령어 또는 에이전트에 의해 호출되는 워크플로우 정의입니다. 예: TDD 워크플로우 등.

### 후크 (Hooks)

도구 이벤트 시 트리거됩니다. 예: `console.log` 감지 시 경고 등.

### 규칙 (Rules)

언제나 준수해야 하는 가이드라인입니다. `common/`과 언어별 디렉토리로 구성됩니다.

상세 내용은 [`rules/README.md`](rules/README.md)를 참조하십시오.

***

## 🗺️ 어떤 에이전트를 사용해야 하나요?

| 원하시는 작업 | 명령어 | 에이전트 |
|--------------|-----------------|------------|
| 신규 기능 계획 | `/everything-claude-code:plan "Add auth"` | planner |
| 시스템 아키텍처 설계 | `/plan` + architect 에이전트 | architect |
| 테스트 우선 구현 | `/tdd` | tdd-guide |
| 코드 품질 리뷰 | `/code-review` | code-reviewer |
| 빌드 오류 수정 | `/build-fix` | build-error-resolver |
| 엔드투엔드 테스트 | `/e2e` | e2e-runner |
| 보안 취약점 점검 | `/security-scan` | security-reviewer |
| 불필요한 코드 제거 | `/refactor-clean` | refactor-cleaner |
| 문서 업데이트 | `/update-docs` | doc-updater |

전체 목록 및 명령어 사용법은 저장소의 에이전트 섹션을 참조하십시오.

***

## ❓ 자주 묻는 질문 (FAQ)

<details>
<summary><b>설치된 에이전트/명령어를 어떻게 확인하나요?</b></summary>

```bash
/plugin list everything-claude-code@everything-claude-code
```
</details>

<details>
<summary><b>후크가 작동하지 않거나 중복 오류가 납니다.</b></summary>
Claude Code v2.1+는 플러그인 후크를 자동 로드하므로 `plugin.json`에 명시적으로 추가하지 마십시오.
</details>

<details>
<summary><b>컨텍스트 윈도우가 너무 빨리 줄어듭니다.</b></summary>
너무 많은 MCP 서버를 활성화하면 토큰을 많이 소비합니다. 프로젝트별로 사용하지 않는 MCP를 비활성화하십시오.
</details>

***
