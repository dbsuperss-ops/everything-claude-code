---
name: projects
description: 등록된 프로젝트 목록 및 각 프로젝트별 본능(instincts) 통계 데이터를 표시합니다.
command: true
---

# 프로젝트 (Projects) 명령어

Continuous Learning v2 시스템에 등록된 프로젝트 목록과 각 프로젝트의 본능 수 및 관찰 횟수(Observations)를 요약하여 표시합니다.

## 실행 방식

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" projects
```

만약 `CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우(수동 설치 등):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py projects
```

## 사용법

```bash
/projects
```

## 상세 처리 내용

1. `~/.claude/homunculus/projects.json` 파일을 읽습니다.
2. 각 프로젝트별로 다음 정보를 표시합니다:
   * 프로젝트 이름, ID, 루트 디렉토리 경로, Git 원격 저장소 주소
   * 개인적으로 습득한 본능과 상속된 본능의 개수
   * 관찰(Observation) 이벤트 누적 횟수
   * 마지막 활동 시각 (Timestamp)
3. 전체 시스템의 **전역 본능 총합**을 함께 출력합니다.

**핵심**: Projects 명령어를 통해 에이전트가 어떤 프로젝트에서 얼마나 많은 지식을 학습했는지 한눈에 파악하고, 여러 개발 환경을 통합적으로 관리할 수 있습니다.
