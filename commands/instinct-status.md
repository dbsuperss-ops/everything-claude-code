---
이름: instinct-status
설명: 학습된 본능(프로젝트 + 글로벌)을 신뢰도와 함께 표시합니다.
명령어 사용 가능: true
---

# Instinct Status 명령어 (본능 상태 확인)

현재 프로젝트에서 학습된 본능과 글로벌 본능을 도메인별로 묶어서 보여줍니다.

## 구현 방법

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" status
```

`CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우 (수동 설치 시):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py status
```

## 사용법

```
/instinct-status
```

## 처리 단계

1. 현재 프로젝트 컨텍스트(git remote/경로 해시) 감지
2. `~/.claude/homunculus/projects/<project-id>/instincts/`에서 프로젝트 본능 읽기
3. `~/.claude/homunculus/instincts/`에서 글로벌 본능 읽기
4. 우선순위 규칙에 따라 병합 (ID 충돌 시 프로젝트 본능이 글로벌보다 우선)
5. 도메인별로 그룹화하여 신뢰도 바(bar) 및 관찰 통계와 함께 표시

## 출력 형식 예시

```
============================================================
  본능 상태 (INSTINCT STATUS) - 총 12개
============================================================

  프로젝트: my-app (a1b2c3d4e5f6)
  프로젝트 범위 본능: 8개
  글로벌 범위 본능: 4개

## 프로젝트 권한 (my-app)
  ### 워크플로우 (3)
    ███████░░░  70%  grep-before-edit [project]
               트리거: 코드를 수정할 때

## 글로벌 (모든 프로젝트에 적용)
  ### 보안 (2)
    █████████░  85%  validate-user-input [global]
               트리거: 사용자 입력을 처리할 때
```
