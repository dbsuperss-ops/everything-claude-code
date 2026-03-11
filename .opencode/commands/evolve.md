---
description: 본능(Inst instincts)을 분석하고 진화된 구조 제안 또는 생성
agent: build
---

# 진화 명령 (Evolve Command)

continuous-learning-v2의 본능(Instincts)을 분석하고 진화시킵니다: $ARGUMENTS

## 임무

다음 명령을 실행하십시오:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" evolve $ARGUMENTS
```

만약 `CLAUDE_PLUGIN_ROOT`를 사용할 수 없다면, 다음 명령을 사용하십시오:

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve $ARGUMENTS
```

## 지원되는 인자 (v2.1)

- 인자 없음: 분석만 수행
- `--generate`: `evolved/{skills,commands,agents}` 아래에 파일도 함께 생성

## 동작 참고 사항

- 분석을 위해 프로젝트 및 글로벌 본능을 사용합니다.
- 트리거 및 도메인 클러스터링을 기반으로 스킬/명령어/에이전트 후보를 보여줍니다.
- 프로젝트 레벨에서 글로벌 레벨로 승격(Promotion) 가능한 후보를 보여줍니다.
- `--generate` 사용 시 출력 경로는 다음과 같습니다:
  - 프로젝트 컨텍스트: `~/.claude/homunculus/projects/<project-id>/evolved/`
  - 글로벌 폴백(Fallback): `~/.claude/homunculus/evolved/`
