---
description: 현재 코드베이스의 에이전트 도구 모음(Harness) 설정을 감사하고 우선순위가 포함된 스코어카드를 반환합니다.
---

# 도구 모음 감사 (Harness Audit) 명령어

현재 저장소의 에이전트 도구 모음 설정을 감사하고 개선을 위한 우선순위 지표를 제공합니다.

## 사용법

`/harness-audit [범위] [--format text|json]`

* `범위` (선택 사항): `repo` (기본값), `hooks`, `skills`, `commands`, `agents`
* `--format`: 출력 형식 지정 (`text` 기본값, `json`은 자동화용)

## 평가 항목

각 항목을 0점에서 10점 사이로 점수화합니다:

1. **도구 커버리지 (Tool Coverage)**: 에이전트가 필요한 도구를 충분히 갖췄는가?
2. **컨텍스트 효율 (Context Efficiency)**: 토큰 소비 대비 정보량이 효율적인가?
3. **품질 관문 (Quality Gates)**: 코드 품질 검증 절차가 견고한가?
4. **기억 유지 (Memory Persistence)**: 세션 간 컨텍스트가 잘 저장되는가?
5. **평가 커버리지 (Eval Coverage)**: 기능 검증을 위한 평가 체계가 갖춰졌는가?
6. **보안 가드레일 (Security Guardrails)**: 위험한 작업에 대한 방어책이 있는가?
7. **비용 효율 (Cost Efficiency)**: 모델 호출 비용이 최적화되어 있는가?

## 결과 보고 형식

다음 내용을 포함하여 결과를 반환합니다:

1. **전체 점수** (총점 70점 만점)
2. **카테고리별 점수** 및 구체적인 발견 사항
3. **최우선 조치 사항 Top 3** (수정해야 할 정확한 파일 경로 포함)
4. 다음 작업에 권장되는 **ECC 스킬** 제안

## 주요 체크리스트

* `hooks/hooks.json`, `scripts/hooks/` 및 후크 테스트 코드를 점검합니다.
* `skills/`, 명령어 목록, 에이전트 정의의 완성도를 확인합니다.
* `.cursor/`, `.opencode/`, `.antigravity/` 등 다양한 도구 간 설정의 일관성을 검증합니다.
* 깨졌거나 오래된 참조(Reference)를 식별하여 표시합니다.

## 결과 예시

```text
도구 모음 감사 결과 (repo): 52/70
- 품질 관문: 9/10
- 평가 커버리지: 6/10
- 비용 효율: 4/10

최우선 조치 사항:
1) 비용 추적 후크 추가: scripts/hooks/cost-tracker.js
2) pass@k 문서 및 템플릿 보완: skills/eval-harness/SKILL.md
3) /harness-audit 명령어 호환성 확보: .antigravity/commands/
```

**핵심**: Harness Audit은 에이전트가 최상의 성능을 낼 수 있는 환경인지 객관적으로 진단합니다. 정기적인 감사를 통해 에이전트의 개발 생산성을 유지하십시오.
