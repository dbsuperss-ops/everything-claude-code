---
description: 학습된 본능(프로젝트 + 글로벌)을 신뢰도와 함께 표시
agent: build
---

# 본능 상태 명령 (Instinct Status Command)

continuous-learning-v2의 본능 상태를 표시합니다: $ARGUMENTS

## 임무

다음 명령을 실행하십시오:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" status
```

만약 `CLAUDE_PLUGIN_ROOT`를 사용할 수 없다면, 다음 명령을 사용하십시오:

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 동작 참고 사항

- 출력에는 프로젝트 범위 및 글로벌 본능이 모두 포함됩니다.
- ID가 충돌하는 경우 프로젝트 본능이 글로벌 본능보다 우선합니다.
- 출력은 도메인별로 그룹화되며 신뢰도 막대(Confidence bars)와 함께 표시됩니다.
- v2.1 버전에서 이 명령은 추가 필터를 지원하지 않습니다.
