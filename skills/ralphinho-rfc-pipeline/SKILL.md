---
name: ralphinho-rfc-pipeline
description: 품질 게이트(Quality Gate), 머지 큐 및 작업 단위 오케스트레이션을 포함한 RFC 주도의 다중 에이전트 DAG 실행 패턴 가이드입니다.
origin: ECC
---

# 랄피뉴 RFC 파이프라인 (Ralphinho RFC Pipeline)

[humanplane](https://github.com/humanplane) 스타일의 RFC 분해 패턴과 다중 작업 단위 오케스트레이션 워크플로우에서 영감을 받았습니다. 한 번에 처리하기에는 너무 큰 기능을 독립적으로 검증 가능한 여러 작업 단위로 나누어 실행할 때 이 스킬을 사용하십시오.

## 파이프라인 단계 (Pipeline Stages)

1. RFC 접수
2. DAG(방향성 비순환 그래프) 분해
3. 작업 단위 할당
4. 작업 단위 구현
5. 작업 단위 검증
6. 머지 큐 및 통합
7. 최종 시스템 확인

## 작업 단위 명세 (Unit Spec)

각 작업 단위는 다음을 포함해야 합니다:
- `id`: 고유 식별자
- `depends_on`: 선행 작업 단위
- `scope`: 범위
- `acceptance_tests`: 인수 테스트
- `risk_level`: 위험 수준
- `rollback_plan`: 롤백 계획

## 복잡도 등급 (Complexity Tiers)

- **Tier 1**: 개별 파일 편집, 결정론적 테스트.
- **Tier 2**: 여러 파일에 걸친 동작 변경, 중간 수준의 통합 위험.
- **Tier 3**: 스키마/인증/성능/보안 관련 변경.

## 작업 단위별 품질 파이프라인

1. 조사(Research)
2. 구현 계획(Implementation Plan)
3. 구현(Implementation)
4. 테스트(Tests)
5. 리뷰(Review)
6. 머지 준비 완료 보고서 작성

## 머지 큐 규칙 (Merge Queue Rules)

- 해결되지 않은 선행 작업 실패가 있는 경우 절대 머지하지 마십시오.
- 항상 최신 통합 브랜치를 기준으로 리베이스(Rebase)하십시오.
- 각 머지 후에 통합 테스트를 다시 실행하십시오.

## 장애 복구 (Recovery)

작업 단위가 정체될 경우: 활성 큐에서 제거하고 조사 내용을 스냅샷으로 남긴 뒤, 범위를 좁혀 다시 시도하십시오.

**결과물**: RFC 실행 로그, 작업 단위 스코어카드, 의존성 그래프 스냅샷, 통합 위험 요약 보고서.

**기억하십시오**: 큰 문제를 작고 관리 가능한 단위로 나누는 것이 대규모 에이전트 워크플로우의 핵심입니다.
    
