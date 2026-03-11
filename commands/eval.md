# Eval 명령어 (Eval Command)

평가(Eval) 주도 개발 워크플로우를 관리합니다.

## 사용법

`/eval [define|check|report|list] [기능-이름]`

## 평가 정의 (Define Evals)

`/eval define 기능-이름`

새로운 평가 정의를 생성합니다:

1. 다음 템플릿으로 `.claude/evals/기능-이름.md` 파일을 생성합니다:

```markdown
## 평가(EVAL): 기능-이름
생성일: $(date)

### 기능 평가 (Capability Evals)
- [ ] [기능 1에 대한 설명]
- [ ] [기능 2에 대한 설명]

### 회귀 평가 (Regression Evals)
- [ ] [기존 동작 1이 여전히 작동함]
- [ ] [기존 동작 2가 여전히 작동함]

### 합격 기준 (Success Criteria)
- 기능 평가에 대해 pass@3 > 90%
- 회귀 평가에 대해 pass^3 = 100%
```

2. 사용자에게 구체적인 기준을 채우도록 요청합니다.

## 평가 확인 (Check Evals)

`/eval check 기능-이름`

특정 기능에 대한 평가를 실행합니다:

1. `.claude/evals/기능-이름.md`에서 평가 정의를 읽습니다.
2. 각 기능 평가에 대해:
   - 기준 검증을 시도합니다.
   - 합격/불합격(PASS/FAIL)을 기록합니다.
   - `.claude/evals/기능-이름.log`에 시도 내용을 기록합니다.
3. 각 회귀 평가에 대해:
   - 관련 테스트를 실행합니다.
   - 기준선(Baseline)과 비교합니다.
   - 합격/불합격(PASS/FAIL)을 기록합니다.
4. 현재 상태를 보고합니다:

```
평가 확인: 기능-이름
========================
기능 평가: Y개 중 X개 합격
회귀 평가: Y개 중 X개 합격
상태: 진행 중(IN PROGRESS) / 준비 완료(READY)
```

## 평가 보고 (Report Evals)

`/eval report 기능-이름`

종합 평가 보고서를 생성합니다:

```
평가 보고서: 기능-이름
=========================
생성일: $(date)

기능 평가 (CAPABILITY EVALS)
----------------
[eval-1]: 합격 (PASS) (pass@1)
[eval-2]: 합격 (PASS) (pass@2) - 재시도 필요함
[eval-3]: 불합격 (FAIL) - 노트 참조

회귀 평가 (REGRESSION EVALS)
----------------
[test-1]: 합격 (PASS)
[test-2]: 합격 (PASS)
[test-3]: 합격 (PASS)

지표 (METRICS)
-------
기능 평가 pass@1: 67%
기능 평가 pass@3: 100%
회귀 평가 pass^3: 100%

노트 (NOTES)
-----
[이슈, 엣지 케이스 또는 관찰 사항]

권고 사항 (RECOMMENDATION)
--------------
[출시 가능(SHIP) / 추가 작업 필요(NEEDS WORK) / 차단됨(BLOCKED)]
```

## 평가 목록 (List Evals)

`/eval list`

모든 평가 정의를 표시합니다:

```
평가 정의 목록
================
feature-auth      [5개 중 3개 합격] 진행 중
feature-search    [5개 중 5개 합격] 준비 완료
feature-export    [4개 중 0개 합격] 시작되지 않음
```

## 인자 (Arguments)

$인자:
- `define <이름>` - 새로운 평가 정의 생성
- `check <이름>` - 평가 실행 및 확인
- `report <이름>` - 전체 보고서 생성
- `list` - 모든 평가 표시
- `clean` - 오래된 평가 로그 삭제 (최근 10회 실행분 유지)
