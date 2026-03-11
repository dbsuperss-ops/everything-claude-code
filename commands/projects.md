---
이름: projects
설명: 알려진 프로젝트 목록과 각 프로젝트별 본능(instinct) 통계를 표시합니다.
명령어 사용 가능: true
---

# Projects 명령어 (프로젝트 목록 확인)

continuous-learning-v2 엔진에서 관리하는 프로젝트 레지스트리 항목과 각 프로젝트별 본능/관찰 횟수를 나열합니다.

## 구현 방법

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" projects
```

`CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우 (수동 설치 시):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py projects
```

## 사용법

```bash
/projects
```

## 처리 단계

1. `~/.claude/homunculus/projects.json` 파일을 읽습니다.
2. 각 프로젝트에 대해 다음 정보를 표시합니다:
   - 프로젝트 이름, ID, 루트 경로, 원격 저장소(remote)
   - 개인(Personal) 및 상속된(Inherited) 본능의 개수
   - 관찰 이벤트 횟수
   - 마지막 활동 시간
3. 글로벌 본능의 총합도 함께 표시합니다.
