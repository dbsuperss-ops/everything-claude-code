---
name: verification-loop
description: "Claude Code 세션의 전체 기능을 점검하는 종합 검증 시스템입니다."
origin: ECC
---

# 검증 사이클 스킬 (Verification Loop)

Claude Code 세션을 위한 종합적인 코드 검증 시스템입니다.

## 적용 시점

다음과 같은 상황에서 이 스킬을 호출하십시오:

* 기능 구현이나 대규모 코드 변경을 완료했을 때
* PR(Pull Request)을 생성하기 직전
* 품질 기준(Quality Gates)을 통과했는지 확인하고 싶을 때
* 리팩토링 수행 후

## 검증 단계

### 1단계: 빌드 검증 (Build)
```bash
npm run build 2>&1 | tail -20
```
빌드에 실패하면 즉시 중단하고 수정하십시오.

### 2단계: 타입 체크 (Type Check)
```bash
# TypeScript 프로젝트
npx tsc --noEmit 2>&1 | head -30

# Python 프로젝트
pyright . 2>&1 | head -30
```
모든 타입 에러를 보고하고, 주요 에러는 즉시 수정하십시오.

### 3단계: 코드 컨벤션 체크 (Lint)
```bash
npm run lint 2>&1 | head -30
```

### 4단계: 테스트 및 커버리지 (Tests & Coverage)
```bash
npm run test -- --coverage 2>&1 | tail -50
```
* **목표**: 최소 80% 이상의 테스트 커버리지 달성 여부 확인

### 5단계: 보안 및 로그 스캔 (Security & Clean)
* 하드코딩된 API 키(`sk-` 등)가 노출되었는지 확인합니다.
* 프로덕션 코드에 남겨진 `console.log`가 있는지 확인합니다.

### 6단계: 변경 사항 검토 (Diff Review)
```bash
git diff --stat
```
변경된 파일들을 하나씩 훑어보며 다음을 체크합니다:
* 실수로 변경된 부분이 없는가?
* 에러 처리가 누락된 곳은 없는가?
* 잠재적인 경계 조건(Edge cases)이 고려되었는가?

---

## 검증 보고서 템플릿 (Verification Report)

모든 단계를 마친 후 다음과 같은 양식으로 보고하십시오:

```
[검증 보고서]
==================
빌드 결과:     [PASS/FAIL]
타입 체크:     [PASS/FAIL] (에러 수: X)
린트(Lint):    [PASS/FAIL] (경고 수: X)
테스트 결과:   [PASS/FAIL] (성공/전체, 커버리지 Z%)
보안/로그:     [PASS/FAIL] (발견된 이슈: X)
변경 파일 수:   [X]

최종 판정:     [PR 준비 완료 / 추가 수정 필요]

수정 필요 사항:
1. ...
2. ...
```

**핵심**: 작은 기능 구현 후나 작업 전환 전, 정기적으로 `/verify`를 실행하여 사이클을 유지하십시오. 빠른 피드백이 실수를 줄이는 가장 좋은 방법입니다.
