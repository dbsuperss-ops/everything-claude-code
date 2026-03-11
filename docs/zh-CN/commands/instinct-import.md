---
name: instinct-import
description: 파일 또는 URL에서 본능(instincts)을 프로젝트 또는 전역 범위로 가져오기(Import)합니다.
command: true
---

# 본능 가져오기 (Instinct Import) 명령어

## 실행 방식

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" import <파일-또는-URL> [--dry-run] [--force] [--min-confidence 0.7] [--scope project|global]
```

만약 `CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우(수동 설치 등):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py import <파일-또는-URL>
```

로컬 파일 경로나 HTTP(S) URL로부터 본능 데이터를 가져올 수 있습니다.

## 사용법

```text
/instinct-import team-instincts.yaml
/instinct-import https://github.com/org/repo/instincts.yaml
/instinct-import team-instincts.yaml --dry-run
/instinct-import team-instincts.yaml --scope global --force
```

## 처리 절차

1. 대상 본능 파일(로컬 경로 또는 URL)을 확보합니다.
2. 파일 형식을 파싱하고 형식을 검증합니다.
3. 기존에 저장된 본능과 중복 여부를 확인합니다.
4. 새로운 본능을 추가하거나 기존 본능과 병합(Merge)합니다.
5. 상속된 본능 디렉토리에 저장합니다:
   * 프로젝트 범위: `~/.claude/homunculus/projects/<프로젝트-id>/instincts/inherited/`
   * 전역 범위: `~/.claude/homunculus/instincts/inherited/`

## 실행 과정 (예시)

```text
📥 가져오기 시도 중: team-instincts.yaml
================================================

12개의 본능이 발견되었습니다.
충돌 여부를 분석 중입니다...

## 🆕 신규 본능 (8개)
- use-zod-validation (신뢰도: 0.7)
- prefer-named-exports (신뢰도: 0.65)
- ...

## ⚠️ 중순 본능 (3개)
이미 유사한 본능이 존재합니다:
  - prefer-functional-style
    (로컬: 0.8 신뢰도, 12회 관찰됨)
    (가져오기: 0.7 신뢰도)
    → 로컬 유지 (신뢰도가 더 높음)

  - test-first-workflow
    (로컬: 0.75 신뢰도)
    (가져오기: 0.9 신뢰도)
    → 가져오기 본능으로 업데이트 (신뢰도가 더 높음)

신규 8개 추가, 1개 업데이트를 진행하시겠습니까?
```

## 병합(Merge) 규칙

동일한 ID를 가진 본능을 가져올 때:
* 가져오는 데이터의 신뢰도가 더 높으면 업데이트 후보가 됩니다.
* 기존 데이터의 신뢰도가 더 높거나 같으면 해당 항목은 건너뜁니다.
* `--force` 옵션을 사용하지 않는 한, 사용자에게 항상 확인을 요청합니다.

## 매개변수 (Flags)

* `--dry-run`: 실제 적용 없이 미리보기만 수행
* `--force`: 확인 절차 없이 즉시 적용
* `--min-confidence <숫자>`: 일정 기준 이상의 신뢰도를 가진 본능만 선별하여 가져옴
* `--scope <project|global>`: 가져올 대상 범위 지정 (기본값: `project`)

**핵심**: 외부의 검증된 본능 데이터를 가져와 에이전트의 지식을 즉각적으로 강화하고, 팀 내 베스트 프랙티스를 손쉽게 확산시킬 수 있습니다.
