---
name: continuous-agent-loop
description: 품질 게이트, 평가 및 복구 제어 기능을 갖춘 자율 에이전트 루프 패턴입니다.
origin: ECC
---

# 지속적 에이전트 루프 (Continuous Agent Loop)

v1.8+ 리전의 표준 루프 스킬 이름입니다. `autonomous-loops`를 대체하지만, 한시적으로 하위 호환성을 유지합니다.

## 루프 선택 흐름

```text
시작
  |
  +-- 엄격한 CI/PR 제어가 필요한가? -- 예 --> continuous-pr
  |                                    
  +-- RFC 분해(Decomposition)가 필요한가? -- 예 --> rfc-dag
  |
  +-- 탐색적 병렬 생성이 필요한가? -- 예 --> infinite
  |
  +-- 아니오(기본값) --> sequential (순차적 루프)
```

## 결합 패턴

권장되는 운영 환경 스택:
1. RFC 분해 (`ralphinho-rfc-pipeline`)
2. 품질 게이트 (`plankton-code-quality` + `/quality-gate`)
3. 평가 루프 (`eval-harness`)
4. 세션 유지 (`nanoclaw-repl`)

## 실패 유형 (Failure Modes)

- 측정 가능한 진전 없는 루프 맴돌기 (Churn)
- 동일한 원인에 대한 반복적인 재시도
- 머지 큐(Merge queue) 중단
- 제어되지 않는 확장으로 인한 비용 급증

## 복구 방법 (Recovery)

- 루프 동결 (Freeze loop)
- `/harness-audit` 명령어 실행
- 실패한 단위로 범위(Scope) 축소
- 명시적인 수락 기준(Acceptance criteria)과 함께 다시 시도
    
