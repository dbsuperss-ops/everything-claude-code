---
name: promote
description: 특정 프로젝트 범위의 본능(instincts)을 모든 프로젝트에서 상시 적용되도록 전역(Global) 범위로 승격시킵니다.
command: true
---

# 승격 (Promote) 명령어

프로젝트 한정적인 본능을 범용적인 지식으로 승격시켜, 모든 개발 환경에서 동일한 에이전트 행동 지침이 적용되도록 합니다.

## 실행 방식

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" promote [본능ID] [--force] [--dry-run]
```

만약 `CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우(수동 설치 등):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py promote [본능ID] [--force] [--dry-run]
```

## 사용법

```bash
/promote                      # 승격 가능한 본능 자동 감지
/promote --dry-run            # 승격 대상 미리보기
/promote --force              # 확인 절차 없이 모든 대상 즉시 승격
/promote grep-before-edit     # 현재 프로젝트의 특정 본능 하나를 지목하여 승격
```

## 상세 처리 프로세스

1. 현재 프로젝트의 컨텍스트를 분석합니다.
2. 특정 `본능ID`가 입력된 경우, 해당 프로젝트 내에 존재하는지 확인 후 즉시 승격시킵니다.
3. ID가 생략된 경우, 다음 기준에 부합하는 본능을 자동으로 탐색합니다:
   * **최소 2개 이상의 프로젝트**에서 공통으로 발견된 패턴
   * **신뢰도(Confidence)**가 일정 기준 이상으로 높은 경우
4. 승격된 본능은 전역 개인 본능 디렉토리(`~/.claude/homunculus/instincts/personal/`)에 저장되며, 속성이 `scope: global`로 변경됩니다.

**핵심**: Promote 명령어를 통해 프로젝트별로 파편화된 경험을 통합된 핵심 역량으로 축적하고, 에이전트의 범용 지능을 한 단계 높일 수 있습니다.
