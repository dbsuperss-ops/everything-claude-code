---
name: plankton-code-quality
description: Plankton을 사용한 작성 시점(Work-time) 코드 품질 강제 적용 가이드입니다. 후기 도구 사용(PostToolUse) 훅을 통해 파일 편집 시마다 자동 포맷팅, 린팅 및 Claude 기반의 자동 수정을 수행합니다.
origin: community
---

# Plankton 코드 품질 스킬

Plankton(@alxfazio 제작)은 Claude Code를 위한 작성 시점 코드 품질 강제 적용 시스템입니다. Plankton은 `PostToolUse` 훅을 통해 파일이 편집될 때마다 포맷터와 린터를 실행하며, 에이전트가 놓친 위반 사항을 수정하기 위해 Claude 하위 프로세스를 생성합니다.

## 적용 시점

* (커밋 시점이 아닌) 파일 편집 시마다 자동 포맷팅과 린팅을 원할 때
* 에이전트가 코드를 수정하는 대신 린트 설정을 변조하여 검사를 통과하려는 행위를 방지하고 싶을 때
* 수정을 위한 계층형 모델 라우팅(단순 스타일은 Haiku, 로직은 Sonnet, 타입은 Opus)이 필요할 때
* 다양한 언어(Python, TypeScript, Shell, YAML, JSON, TOML, Markdown, Dockerfile)를 혼용할 때

## 작동 원리

### 3단계 아키텍처 (Three-Phase Architecture)

Claude Code가 파일을 편집하거나 작성할 때마다 Plankton의 `multi_linter.sh` (PostToolUse 훅)가 실행됩니다:

1. **1단계: 자동 포맷팅 (정적)**
   * 포맷터 실행(ruff format, biome, shfmt, taplo, markdownlint 등)
   * 문제의 40~50%를 자동으로 조용히 수정
   * 메인 에이전트에게는 출력을 보내지 않음

2. **2단계: 위반 사항 수집 (JSON)**
   * 린터를 실행하고 수정 불가능한 위반 사항 수집
   * 구조화된 JSON 반환: `{line, column, code, message, linter}`
   * 여전히 메인 에이전트에게는 직접 보고하지 않음

3. **3단계: 위임 및 검증 (Delegate + Verify)**
   * 위반 사항 JSON을 가지고 `claude -p` 하위 프로세스 생성
   * 위반 복잡도에 따라 모델 라우팅:
     * **Haiku**: 포맷팅, 임포트, 스타일 등 (120초 제한)
     * **Sonnet**: 복잡도, 리팩토링 (300초 제한)
     * **Opus**: 타입 시스템, 깊은 추론 (600초 제한)
   * 수정 확인을 위해 1~2단계를 재실행
   * 깨끗하면 종료 코드 0, 위반 사항이 남으면 종료 코드 2 (메인 에이전트에 보고)

### 메인 에이전트가 보는 화면

| 시나리오 | 에이전트 알림 | 훅 종료 코드 |
|----------|-----------|-----------|
| 위반 사항 없음 | 없음 | 0 |
| 하위 프로세스가 모두 수정 | 없음 | 0 |
| 수정 후에도 위반 사항 남음 | `[hook] N violation(s) remain` | 2 |
| 권장 경고 (중복, 구형 도구 등) | `[hook:advisory] ...` | 0 |

## 설정 및 설치

### 빠른 시작

```bash
# Plankton 클론 (개발자: @alxfazio)
git clone https://github.com/alexfazio/plankton.git
cd plankton

# 핵심 의존성 설치
brew install jaq ruff uv

# Python 린터 설치
uv sync --all-extras

# Claude Code 실행 — 훅이 자동으로 활성화됨
claude
```

별도의 설치 명령이나 플러그인 설정이 필요 없습니다. Claude Code를 실행하면 `.claude/settings.json`의 훅 설정이 Plankton 디렉토리에서 자동으로 로드됩니다.

## ECC와 함께 사용하기

### 상호 보완 관계

| 기능 | ECC | Plankton |
|---------|-----|----------|
| **코드 품질 강제** | PostToolUse 훅 (Prettier, tsc) | PostToolUse 훅 (20+ 린터 + 자동 수정) |
| **보안 스캔** | AgentShield, security-reviewer | Bandit (Python), Semgrep (TS/JS) |
| **설정 보호** | - | PreToolUse 차단 + Stop 훅 감지 |
| **패키지 관리자** | 감지 및 설정 | 강제 적용 (구형 관리자 차단) |
| **모델 라우팅** | 수동 (`/model opus`) | 자동 (위반 복잡도 기반 계층화) |

### 권장 조합 전략

1. **ECC**를 플러그인(에이전트, 스킬, 명령, 규칙)으로 설치합니다.
2. **Plankton** 훅을 추가하여 실시간 코드 품질을 강제합니다.
3. **AgentShield**를 사용하여 보안 감사를 수행합니다.
4. PR(Pull Request) 전 마지막 관문으로 ECC의 **verification-loop**를 실행합니다.

## 설정 구성 (Config Reference)

Plankton의 `.claude/hooks/config.json`에서 모든 동작을 제어할 수 있습니다:

* `languages`: 사용하지 않는 언어는 비활성화하여 훅 실행 속도를 높이십시오.
* `volume_threshold`: 위반 사항이 일정 개수를 넘으면 자동으로 상위 모델 계층으로 업그레이드합니다.
* `subprocess_delegation`: `false`로 설정하면 자동 수정 없이 위반 사항 보고만 수행합니다.

## 환경 변수 (Environment Variables)

* `HOOK_SKIP_SUBPROCESS=1`: 자동 수정을 건너뛰고 위반 사항만 보고합니다.
* `HOOK_SUBPROCESS_TIMEOUT=N`: 모델 계층별 타임아웃을 재설정합니다.
* `HOOK_SKIP_PM=1`: 패키지 관리자 강제 적용을 우회합니다.

---
**주의**: Plankton은 강력한 코드 품질 도구입니다. 프로젝트의 기술 스택에 맞춰 지원 언어를 최적화하여 최상의 성능을 유지하십시오.
