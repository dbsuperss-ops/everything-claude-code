---
이름: instinct-import
설명: 파일 또는 URL에서 본능(instincts)을 프로젝트/글로벌 범위로 가져옵니다.
명령어 사용 가능: true
---

# Instinct Import 명령어 (본능 가져오기)

## 구현 방법

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" import <파일-또는-URL> [--dry-run] [--force] [--min-confidence 0.7] [--scope project|global]
```

`CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우 (수동 설치 시):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py import <파일-또는-URL>
```

로컬 파일 경로 또는 HTTP(S) URL에서 본능을 가져옵니다.

## 사용법

```
/instinct-import team-instincts.yaml
/instinct-import https://github.com/org/repo/instincts.yaml
/instinct-import team-instincts.yaml --dry-run
/instinct-import team-instincts.yaml --scope global --force
```

## 처리 단계

1. 본능 파일 가져오기 (로컬 경로 또는 URL)
2. 형식 파싱 및 유효성 검사
3. 기존 본능과 중복 여부 확인
4. 새로운 본능 병합 또는 추가
5. 상속된(inherited) 본능 디렉토리에 저장:
   - 프로젝트 범위: `~/.claude/homunculus/projects/<project-id>/instincts/inherited/`
   - 글로벌 범위: `~/.claude/homunculus/instincts/inherited/`

## 가져오기 프로세스 예시

```
📥 본능 가져오는 중: team-instincts.yaml
================================================

가져올 본능 12개를 찾았습니다.

충돌 분석 중...

## 새로운 본능 (8개)
다음을 추가합니다:
   ✓ use-zod-validation (신뢰도: 0.7)
   ✓ prefer-named-exports (신뢰도: 0.65)
   ✓ test-async-functions (신뢰도: 0.8)
   ...

## 중복된 본능 (3개)
이미 유사한 본능이 있습니다:
  ⚠️ prefer-functional-style
     로컬: 0.8 신뢰도, 12회 관찰
     가져오기: 0.7 신뢰도
     → 로컬 유지 (더 높은 신뢰도)

  ⚠️ test-first-workflow
     로컬: 0.75 신뢰도
     가져오기: 0.9 신뢰도
     → 가져온 것으로 업데이트 (더 높은 신뢰도)

새로운 본능 8개 추가, 1개 업데이트하시겠습니까?
```

## 병합 규칙 (Merge Behavior)

이미 존재하는 ID의 본능을 가져올 때:
- 더 높은 신뢰도의 본능이 업데이트 후보가 됩니다.
- 동일하거나 낮은 신뢰도의 본능은 건너뜁니다.
- `--force` 플래그를 사용하지 않는 한 사용자의 확인을 거칩니다.

## 출처 추적 (Source Tracking)

가져온 본능은 다음 정보와 함께 저장됩니다:
```yaml
source: inherited
scope: project
imported_from: "team-instincts.yaml"
project_id: "a1b2c3d4e5f6"
project_name: "my-project"
```

## 플래그 (Flags)

- `--dry-run`: 실제 가져오기 없이 미리보기만 수행
- `--force`: 확인 프롬프트 생략
- `--min-confidence <n>`: 임계값 이상의 신뢰도를 가진 본능만 가져옴
- `--scope <project|global>`: 대상 범위 선택 (기본값: `project`)

## 출력 결과

가져오기 완료 후:
```
✅ 가져오기 완료!

추가됨: 본능 8개
업데이트됨: 본능 1개
건너뜀: 본능 3개 (동일하거나 높은 신뢰도가 이미 존재함)

새로운 본능이 저장된 위치: ~/.claude/homunculus/instincts/inherited/

/instinct-status를 실행하여 모든 본능을 확인하세요.
```
