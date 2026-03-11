---
name: instinct-status
description: 학습된 본능(instincts)의 목록 및 신뢰도를 프로젝트 및 전역 범위별로 표시합니다.
command: true
---

# 본능 상태 (Instinct Status) 명령어

현재 프로젝트에서 학습된 본능과 전역 본능을 도메인별로 그룹화하여 표시합니다.

## 실행 방식

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" status
```

만약 `CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우(수동 설치 등):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 사용법

```text
/instinct-status
```

## 처리 절차

1. 현재 프로젝트 컨텍스트(Git 원격 저장소 또는 경로 해시)를 감지합니다.
2. 프로젝트 단위 본능을 읽어옵니다: `~/.claude/homunculus/projects/<프로젝트-id>/instincts/`
3. 전역 단위 본능을 읽어옵니다: `~/.claude/homunculus/instincts/`
4. 본능을 병합하고 우선순위 규칙을 적용합니다 (ID 충돌 시 프로젝트 단위 본능이 전역 본능을 덮어씁니다).
5. 도메인별로 그룹화하여 신뢰도 막대 그래프 및 관찰 통계와 함께 표시합니다.

## 출력 데이터 형식 (예시)

```text
============================================================
  본능 상태 리포트 - 총 12개
============================================================

  프로젝트: my-app (a1b2c3d4e5f6)
  프로젝트 본능: 8개
  전역 본능: 4개

## 📁 프로젝트 한정 본능 (my-app)
  ### 워크플로우 (3)
    ███████░░░  70%  grep-before-edit [project]
               트리거: 코드 수정 시

## 🌐 전역 본능 (모든 프로젝트 적용)
  ### 보안 (2)
    █████████░  85%  validate-user-input [global]
               트리거: 사용자 입력 처리 시
```

**핵심**: Instinct Status를 통해 에이전트가 어떤 습관과 지식을 학습했는지 실시간으로 파악할 수 있습니다. 신뢰도가 낮은 본능은 추가적인 학습이나 수동 조정을 통해 강화할 수 있습니다.
