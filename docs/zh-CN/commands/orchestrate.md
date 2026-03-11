---
description: 복잡한 작업을 해결하기 위해 여러 에이전트를 순차적으로 연결하는 워크플로우를 조율(Orchestrate)합니다.
---

# 조율 (Orchestrate) 명령어

복잡하고 단계적인 작업을 위해 전문화된 에이전트들을 하나의 워크플로우로 묶어 실행합니다.

## 사용법

`/orchestrate [워크플로우-유형] [작업-설명]`

## 주요 워크플로우 유형

### feature (기능 구현)
전체 기능을 구현하기 위한 종합 워크플로우:
```text
planner (계획) -> tdd-guide (TDD 구현) -> code-reviewer (품질 검토) -> security-reviewer (보안 검토)
```

### bugfix (버그 수정)
에러 조사 및 수정을 위한 워크플로우:
```text
planner (분석) -> tdd-guide (수정 구현) -> code-reviewer (검증)
```

### refactor (리팩토링)
안전한 코드 구조 개선을 위한 워크플로우:
```text
architect (설계) -> code-reviewer (영향도 분석) -> tdd-guide (재구현)
```

### security (보안 강화)
보안 취약점 점검 및 조치 워크플로우:
```text
security-reviewer (취약점 점검) -> code-reviewer (코드 분석) -> architect (보안 설계)
```

## 에이전트 간 업무 인계(Handoff) 방식

각 단계의 에이전트는 다음 절차를 따릅니다:

1. 이전 에이전트로부터 전달받은 컨텍스트와 함께 **에이전트를 호출**합니다.
2. 수행 결과를 구조화된 **인계 문서(Handoff Document)**로 작성합니다.
3. 인계 문서를 **다음 에이전트에게 전달**합니다.
4. 모든 단계가 완료되면 결과를 최종 보고서로 **취합**합니다.

## 인계 문서(Handoff) 형식

```markdown
## 업무 인계: [보내는 에이전트] -> [받는 에이전트]

### 배경 (Background)
[지금까지 수행된 작업 요약]

### 주요 발견 사항 (Findings)
[핵심 분석 결과 또는 결정 사항]

### 수정된 파일 (Modified Files)
[작업이 반영된 파일 목록]

### 남은 과제 (Pending Issues)
[다음 에이전트가 처리해야 할 미결 사항]

### 권장 조치 (Recommendations)
[다음 단계에서 추천하는 작업 방향]
```

---

## 최종 보고서 (Orchestration Report) 형식

```text
ORCHESTRATION REPORT
====================
워크플로우: feature
작업명: 사용자 인증 추가
에이전트 체인: planner -> tdd-guide -> code-reviewer -> security-reviewer

[요약] 작업 전체 내용에 대한 짧은 요약문

[에이전트별 결과]
- Planner: 계획 수립 완료
- TDD Guide: 테스트 통과 및 구현 완료
- ...

[코드 변경] 수정된 모든 파일 목록
[테스트 결과] 테스트 통과/실패 요약
[보안 상태] 발견된 취약점 및 조치 사항
[최종 권고] 배포 가능(SHIP) / 추가 보완 필요(NEEDS WORK) / 중단(BLOCKED)
```

---

## 매개변수 (Arguments)

* `feature <설명>` - 신규 기능 구현용
* `bugfix <설명>` - 오류 수정용
* `refactor <설명>` - 리팩토링용
* `security <설명>` - 보안 리뷰용
* `custom <에이전트목록> <설명>` - 사용자 정의 에이전트 시퀀스 (예: "architect,code-reviewer")

**핵심**: Orchestrate 명령어는 개별 에이전트의 전문성을 결합하여 대규모 작업을 체계적으로 완성합니다. 특히 결제나 보안 등 민감한 작업에서는 에이전트 체인에 `security-reviewer`를 반드시 포함시키십시오.
