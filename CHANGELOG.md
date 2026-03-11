# 변경 로그 (Changelog)

## 1.8.0 - 2026-03-04

### 주요 변경 사항

- 신뢰성, 평가 규율 및 자율 루프 운영에 중점을 둔 하네스 우선(Harness-first) 릴리스.
- 후크(Hook) 런타임에서 이제 프로필 기반 제어 및 대상 후크 비활성화를 지원합니다.
- NanoClaw v2에서 모델 라우팅, 스킬 최신화(hot-load), 분기, 검색, 압축(compaction), 내보내기 및 지표 기능이 추가되었습니다.

### 코어 (Core)

- 새로운 명령어 추가: `/harness-audit`, `/loop-start`, `/loop-status`, `/quality-gate`, `/model-route`.
- 새로운 스킬 추가:
  - `agent-harness-construction`
  - `agentic-engineering`
  - `ralphinho-rfc-pipeline`
  - `ai-first-engineering`
  - `enterprise-agent-ops`
  - `nanoclaw-repl`
  - `continuous-agent-loop`
- 새로운 에이전트 추가:
  - `harness-optimizer`
  - `loop-operator`

### 후크 신뢰성 (Hook Reliability)

- 견고한 폴백 검색을 통해 SessionStart 루트 확인 문제를 수정했습니다.
- 세션 요약 영속화(persistence)를 트랜스크립트 페이로드를 사용할 수 있는 `Stop` 단계로 이동했습니다.
- quality-gate 및 cost-tracker 후크를 추가했습니다.
- 불안정한 인라인 후크 한 줄 코드를 전용 스크립트 파일로 대체했습니다.
- `ECC_HOOK_PROFILE` 및 `ECC_DISABLED_HOOKS` 제어 기능을 추가했습니다.

### 크로스 플랫폼 (Cross-Platform)

- 문서 경고 로직에서 Windows 안전 경로 처리를 개선했습니다.
- 비대화형 정지(hang)를 방지하기 위해 옵저버 루프 동작을 강화했습니다.

### 참고 사항 (Notes)

- `autonomous-loops`는 한 번의 릴리스 동안 호환성 별칭으로 유지됩니다. 정식 명칭은 `continuous-agent-loop`입니다.

### 크레딧 (Credits)

- [zarazhangrui](https://github.com/zarazhangrui)로부터 영감을 받음
- [humanplane](https://github.com/humanplane)의 homunculus에서 영감을 받음
    
