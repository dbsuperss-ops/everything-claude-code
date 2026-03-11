# 오케스트레이션 명령어 (Orchestrate Command)

복잡한 작업을 처리하기 위한 순차적 에이전트 워크플로우입니다.

## 사용법

`/orchestrate [워크플로우-유형] [작업-설명]`

## 워크플로우 유형

### 기능 개발 (feature)
전체 기능 구현 워크플로우:
```
planner (계획가) -> tdd-guide (TDD 가이드) -> code-reviewer (코드 리뷰어) -> security-reviewer (보안 리뷰어)
```

### 버그 수정 (bugfix)
버그 조사 및 수정 워크플로우:
```
planner -> tdd-guide -> code-reviewer
```

### 리팩토링 (refactor)
안전한 리팩토링 워크플로우:
```
architect (아키텍트) -> code-reviewer -> tdd-guide
```

### 보안 (security)
보안 중심 리뷰 워크플로우:
```
security-reviewer -> code-reviewer -> architect
```

## 실행 패턴

워크플로우의 각 에이전트에 대해:

1. 이전 에이전트의 컨텍스트를 담아 **에이전트를 호출**합니다.
2. 결과물을 구조화된 인수인계(handoff) 문서로 **수집**합니다.
3. 체인의 **다음 에이전트에게 전달**합니다.
4. 최종 보고서로 **결과를 취합**합니다.

## 인수인계 문서 형식 (Handoff Document Format)

에이전트 간에 인수인계 문서를 작성합니다:

```markdown
## 인수인계(HANDOFF): [이전-에이전트] -> [다음-에이전트]

### 컨텍스트 (Context)
[수행된 작업 요약]

### 발견 사항 (Findings)
[주요 발견 사항 또는 결정 사항]

### 수정된 파일
[수정된 파일 목록]

### 미해결 질문
[다음 에이전트가 해결해야 할 항목]

### 권고 사항
[제안된 다음 단계]
```

## 예시: 기능 개발 워크플로우 (Feature Workflow)

```
/orchestrate feature "사용자 인증 기능 추가"
```

다음과 같이 실행됩니다:

1. **플래너 에이전트 (Planner Agent)**
   - 요구사항 분석
   - 구현 계획 생성
   - 의존성 식별
   - 출력: `HANDOFF: planner -> tdd-guide`

2. **TDD 가이드 에이전트 (TDD Guide Agent)**
   - 플래너의 인수인계 내용 확인
   - 테스트 코드 우선 작성
   - 테스트 통과를 위한 실제 구현
   - 출력: `HANDOFF: tdd-guide -> code-reviewer`

3. **코드 리뷰어 에이전트 (Code Reviewer Agent)**
   - 구현 내용 리뷰
   - 이슈 점검
   - 개선 사항 제안
   - 출력: `HANDOFF: code-reviewer -> security-reviewer`

4. **보안 리뷰어 에이전트 (Security Reviewer Agent)**
   - 보안 감사
   - 취약점 점검
   - 최종 승인
   - 출력: 최종 보고서 (Final Report)

## 최종 보고서 형식 (Final Report Format)

```
오케스트레이션 보고서 (ORCHESTRATION REPORT)
====================
워크플로우: feature
작업: 사용자 인증 기능 추가
에이전트: planner -> tdd-guide -> code-reviewer -> security-reviewer

요약 (SUMMARY)
-------
[한 단락 요약]

에이전트별 출력 (AGENT OUTPUTS)
-------------
Planner: [요약]
TDD Guide: [요약]
Code Reviewer: [요약]
Security Reviewer: [요약]

변경 파일 (FILES CHANGED)
-------------
[수정된 모든 파일 목록]

테스트 결과 (TEST RESULTS)
------------
[테스트 통과/실패 요약]

보안 상태 (SECURITY STATUS)
---------------
[보안 관련 발견 사항]

최종 권고 (RECOMMENDATION)
--------------
[출시 가능(SHIP) / 추가 작업 필요(NEEDS WORK) / 차단됨(BLOCKED)]
```

## 병렬 실행 (Parallel Execution)

독립적인 점검이 필요한 경우 에이전트를 병렬로 실행합니다:

```markdown
### 병렬 단계 (Parallel Phase)
동시 실행:
- code-reviewer (품질 점검)
- security-reviewer (보안 점검)
- architect (디자인 점검)

### 결과 병합 (Merge Results)
출력 내용을 하나의 보고서로 통합
```

## 인자 (Arguments)

$인자:
- `feature <설명>` - 전체 기능 개발 워크플로우
- `bugfix <설명>` - 버그 수정 워크플로우
- `refactor <설명>` - 리팩토링 워크플로우
- `security <설명>` - 보안 리뷰 워크플로우
- `custom <에이전트-목록> <설명>` - 사용자 정의 에이전트 순서

## 사용자 정의 워크플로우 예시

```
/orchestrate custom "architect,tdd-guide,code-reviewer" "캐싱 레이어 재설계"
```

## 팁 (Tips)

1. 복잡한 기능은 반드시 **플래너(planner)부터 시작**하십시오.
2. 머지(Merge) 전에는 **항상 코드 리뷰어(code-reviewer)를 포함**하십시오.
3. 인증/결제/개인정보 관련 작업에는 **보안 리뷰어(security-reviewer)를 활용**하십시오.
4. 인수인계 문서는 **간결하게 유지**하며 다음 에이전트에게 필요한 내용에 집중하십시오.
5. 필요한 경우 에이전트 사이단계에서 **검증(verification)을 실행**하십시오.
