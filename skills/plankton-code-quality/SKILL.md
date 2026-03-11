---
name: plankton-code-quality
description: Plankton을 사용한 작성 시점(Write-time) 코드 품질 강제 — 하이후(Hook)를 통해 모든 파일 편집 시 자동 포맷팅, 린팅 및 Claude 지원 수정을 수행합니다.
origin: community
---

# Plankton 코드 품질 스킬 (Plankton Code Quality Skill)

Claude Code를 위한 작성 시점 코드 품질 강제 시스템인 Plankton(제작: @alxfazio) 통합 참조 가이드입니다. Plankton은 파일 편집 시마다 `PostToolUse` 후크를 통해 포맷터와 린터를 실행하고, 에이전트가 놓친 위반 사항을 수정하기 위해 Claude 서브프로세스를 가동합니다.

## 활성화 시점

- 커밋 시점이 아닌, 파일 편집 시마다 자동 포맷팅과 린팅을 원할 때
- 에이전트가 코드를 수정하는 대신 린터 설정을 변조하여 통과시키려는 행위를 방지하고 싶을 때
- 수정 작업의 복잡도에 따라 모델을 다르게 라우팅(Haiku: 스타일, Sonnet: 로직, Opus: 타입 등)하고 싶을 때
- 다양한 언어(Python, TS, Shell, YAML, JSON, Markdown 등)를 혼용해 작업할 때

## 작동 원리 (3단계 아키텍처)

1. **Phase 1: 자동 포맷 (Silent)** - `ruff`, `biome`, `shfmt` 등을 실행하여 이슈의 40-50%를 자동으로 조용히 수정합니다.
2. **Phase 2: 위반 사항 수집 (JSON)** - 수정 불가능한 위반 사항을 수집하여 구조화된 JSON으로 반환합니다.
3. **Phase 3: 위임 및 검증 (Delegate + Verify)** - 위반 사항 JSON을 가지고 Claude 서브프로세스를 생성합니다. 위상 상태에 따라 적절한 모델(Haiku, Sonnet, Opus)에 수정을 맡기고, 다시 1-2단계를 실행하여 완벽히 수정되었는지 검증합니다.

## 주요 기능 및 보호 장치

- **에이전트 시점**: 대부분의 품질 문제는 투명하게 해결되며, 서브프로세스조차 해결하지 못한 경우에만 메인 에이전트에게 보고됩니다.
- **설정 파일 보호 (Config Protection)**: 에이전트가 코드를 고치는 대신 `.ruff.toml`이나 `biome.json` 등을 수정하여 규칙을 비활성화하는 것을 3중 레이어(PreToolUse, Stop hook, Protected list)로 차단합니다.
- **패키지 매니저 강제**: `pip` 대신 `uv`, `npm` 대신 `bun` 사용을 강제하는 등 최신 도구 사용을 유도합니다.

## 설정 및 통합

- `git clone`을 통해 Plankton을 프로젝트에 추가하고 `uv`, `ruff`, `biome` 등의 의존성을 설치하십시오.
- `.claude/settings.json`에 후크 설정을 추가하면 자동으로 활성화됩니다.
- **ECC와의 연동**: ECC는 보안 스캔 및 검증 루프를 담당하고, Plankton은 실시간 코드 품질 강제를 담당하도록 적절히 조합하여 사용하십시오.

## 환경 변수 제어

- `HOOK_SKIP_SUBPROCESS=1`: AI 서브프로세스 수정을 건너뛰고 위반 사항만 보고합니다.
- `HOOK_SKIP_PM=1`: 패키지 매니저 강제 기능을 우회합니다.

**기억하십시오**: Plankton은 에이전트가 작성하는 코드의 품질을 실시간으로 감시하고 교정하는 "품질 경찰" 역할을 합니다. 이를 통해 코드베이스가 항상 깨끗하고 표준화된 상태를 유지하도록 돕습니다.
    
