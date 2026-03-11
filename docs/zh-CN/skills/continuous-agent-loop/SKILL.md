---
name: continuous-agent-loop
description: 품질 게이트, 평가 및 복구 제어 기능이 포함된 지속적인 자율 에이전트 루프 패턴입니다.
origin: ECC
---

# 지속적 에이전트 루프 (Continuous Agent Loop)

이 스킬은 v1.8+ 버전의 표준 루프 스킬 명칭입니다. 한 번의 릴리스 주기 동안 호환성을 유지하면서 기존 `autonomous-loops`를 대체합니다.

## 루프 선택 프로세스

```text
시작
  |
  +-- 엄격한 CI/PR 제어가 필요한가? -- 예 --> continuous-pr
  |                                    
  +-- RFC 분해가 필요한가? -- 예 --> rfc-dag
  |
  +-- 탐색적인 병렬 생성이 필요한가? -- 예 --> infinite
  |
  +-- 기본값 --> sequential
```

## 조합 패턴

권장되는 프로덕션 스택:

1. RFC 분해 (`ralphinho-rfc-pipeline`)
2. 품질 게이트 (`plankton-code-quality` + `/quality-gate`)
3. 평가 루프 (`eval-harness`)
4. 세션 유지 (`nanoclaw-repl`)

## 실패 유형 (Failure Modes)

* 측정 가능한 진전 없이 루프가 공회전함
* 동일한 근본 원인으로 인해 반복적으로 재시도함
* 병합 큐(Merge queue) 정체
* 무제한 실행으로 인한 비용 급증

## 복구 방법

* 루프를 일시 중지(Freeze)함
* `/harness-audit` 실행
* 실패한 단위로 범위를 좁힘
* 명확한 인수 조건(Acceptance criteria)을 가지고 재시작
