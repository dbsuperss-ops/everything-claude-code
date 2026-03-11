# 모델 라우팅 명령어 (Model Route Command)

작업의 복잡도와 예산에 맞춰 현재 작업에 가장 적합한 모델 티어(tier)를 추천합니다.

## 사용법

`/model-route [작업-설명] [--budget low|med|high]`

## 라우팅 휴리스틱 (Routing Heuristic)

- `haiku`: 결정론적이고 위험도가 낮은 기계적 변경 작업
- `sonnet`: 구현 및 리팩토링을 위한 기본 모델
- `opus`: 아키텍처 설계, 심층 리뷰, 모호한 요구사항 분석

## 필수 출력 항목

- 권장 모델
- 신뢰 수준
- 해당 모델이 적합한 이유
- 첫 번째 시도 실패 시의 폴백(Fallback) 모델

## 인자 (Arguments)

$인자:
- `[작업-설명]` (선택적 자유 텍스트)
- `--budget low|med|high` (선택 사항)
