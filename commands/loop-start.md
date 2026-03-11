# 루프 시작 명령어 (Loop Start Command)

안전 기본 설정이 적용된 관리형 자율 루프(Autonomous Loop) 패턴을 시작합니다.

## 사용법

`/loop-start [패턴] [--mode safe|fast]`

- `패턴`: `sequential`, `continuous-pr`, `rfc-dag`, `infinite`
- `--mode`:
  - `safe` (기본값): 엄격한 품질 관문(Quality Gate) 및 체크포인트 적용
  - `fast`: 속도를 위해 품질 관문 축소

## 흐름 (Flow)

1. 저장소 상태 및 브랜치 전략을 확인합니다.
2. 루프 패턴 및 모델 티어(tier) 전략을 선택합니다.
3. 선택한 모드에 필요한 후크/프로필을 활성화합니다.
4. 루프 계획을 생성하고 `.claude/plans/` 아래에 런북(Runbook)을 작성합니다.
5. 루프를 시작하고 모니터링하기 위한 명령어를 출력합니다.

## 필수 안전 점검 사항

- 첫 번째 루프 반복 전에 테스트 통과 여부를 확인합니다.
- `ECC_HOOK_PROFILE`이 글로벌하게 비활성화되어 있지 않은지 확인합니다.
- 루프에 명시적인 중단 조건이 있는지 확인합니다.

## 인자 (Arguments)

$인자:
- `<패턴>` (선택 사항): `sequential|continuous-pr|rfc-dag|infinite`
- `--mode safe|fast` (선택 사항)
