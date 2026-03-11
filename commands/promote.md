---
이름: promote
설명: 프로젝트 범위의 본능(instincts)을 글로벌 범위로 승격시킵니다.
명령어 사용 가능: true
---

# Promote 명령어 (글로벌 승격)

continuous-learning-v2 엔진에서 프로젝트 범위의 본능을 글로벌 범위로 승격시킵니다.

## 구현 방법

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" promote [본능-ID] [--force] [--dry-run]
```

`CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우 (수동 설치 시):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py promote [본능-ID] [--force] [--dry-run]
```

## 사용법

```bash
/promote                      # 승격 후보 자동 감지
/promote --dry-run            # 자동 승격 후보 미리보기
/promote --force              # 확인 프롬프트 없이 적격한 모든 후보 승격
/promote grep-before-edit     # 현재 프로젝트에서 특정 본능 하나를 승격
```

## 처리 단계

1. 현재 프로젝트를 감지합니다.
2. `본능-ID`가 제공된 경우, 해당 본능만 승격합니다 (현재 프로젝트에 존재하는 경우).
3. ID가 없는 경우, 다음 조건에 맞는 프로젝트 교차 후보를 찾습니다:
   - 최소 2개 이상의 프로젝트에서 나타남
   - 신뢰도 임계값을 충족함
4. 승격된 본능을 `scope: global` 설정과 함께 `~/.claude/homunculus/instincts/personal/` 경로에 저장합니다.
