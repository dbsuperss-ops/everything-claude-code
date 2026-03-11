---
name: ralphinho-rfc-pipeline
description: RFC 기반의 멀티 에이전트 DAG(Directed Acyclic Graph) 실행 패턴입니다. 품질 게이트, 병합 큐 및 작업 단위 오케스트레이션을 포함합니다.
origin: ECC
---

# Ralphinho RFC 파이프라인

[humanplane](https://github.com/humanplane) 스타일의 RFC 분해 패턴과 멀티 유닛 오케스트레이션 워크플로우에서 영감을 받은 스킬입니다.

단일 에이전트가 처리하기에는 너무 방대하여 독립적이고 검증 가능한 작업 단위(Units)로 분할해야 하는 기능을 개발할 때 이 스킬을 사용하십시오.

## 파이프라인 단계 (Pipeline Stages)

1. RFC 접수 및 검토
2. DAG 분해 (작업 간 의존성 그래프 생성)
3. 단위(Unit) 할당
4. 단위별 구현
5. 단위별 검증
6. 병합 큐(Merge Queue) 및 통합
7. 최종 시스템 검증

## 단위 명세 템플릿 (Unit Spec Template)

각 작업 단위에는 다음 항목이 포함되어야 합니다:

* `id`: 단위 고유 식별자
* `depends_on`: 선행 작업 ID 목록
* `scope`: 작업 범위 설명
* `acceptance_tests`: 수락 테스트 기준
* `risk_level`: 위험 수준 (낮음/중간/높음)
* `rollback_plan`: 롤백 계획

## 복잡도 계층 (Complexity Tiers)

* **계층 1**: 독립된 파일 편집, 결정론적 테스트 가능
* **계층 2**: 여러 파일에 걸친 동작 변경, 중간 정도의 통합 리스크
* **계층 3**: 아키텍처/인증/성능/보안 관련 중대 변경

## 단위별 품질 파이프라인

1. 조사 (Research)
2. 구현 계획 (Implementation Plan)
3. 구현 (Implementation)
4. 테스트 (Test)
5. 리뷰 (Review)
6. 병합 준비 보고 (Merge Readiness Report)

## 병합 큐 규칙 (Merge Queue Rules)

* 해결되지 않은 의존성 문제가 있는 단위는 절대 병합하지 마십시오.
* 단위 브랜치를 항상 최신 통합 브랜치로 리베이스(Rebase)하십시오.
* 큐 병합이 발생할 때마다 통합 테스트를 재실행하십시오.

## 복구 프로세스 (Recovery)

작업 단위가 중단되거나 정체된 경우:
* 활성 큐 프로젝트에서 해당 단위를 제거합니다.
* 현재까지의 발견 사항을 스냅샷으로 기록합니다.
* 범위를 축소하여 단위를 다시 생성합니다.
* 업데이트된 제약 조건을 적용하여 재시도합니다.

## 결과물 (Outputs)

* RFC 실행 로그
* 단위별 스코어카드(Scorecard)
* 의존성 그래프 스냅샷
* 통합 리스크 요약 보고서
