# 하네스 감사 명령어 (Harness Audit Command)

현재 저장소의 에이전트 하네스 설정을 감사하고 우선순위가 지정된 스코어카드를 반환합니다.

## 사용법

`/harness-audit [대상] [--format text|json]`

- `대상` (선택 사항): `repo` (기본값), `hooks`, `skills`, `commands`, `agents`
- `--format`: 출력 스타일 (`text` 기본값, 자동화를 위한 `json`)

## 평가 항목

각 카테고리에 대해 `0`점에서 `10`점 사이로 점수를 매깁니다:

1. 도구 커버리지 (Tool Coverage)
2. 컨텍스트 효율성 (Context Efficiency)
3. 품질 관문 (Quality Gates)
4. 메모리 지속성 (Memory Persistence)
5. 평가 커버리지 (Eval Coverage)
6. 보안 가드레일 (Security Guardrails)
7. 비용 효율성 (Cost Efficiency)

## 출력 규약 (Output Contract)

다음 내용을 반환합니다:

1. `overall_score` (70점 만점)
2. 카테고리별 점수 및 구체적인 발견 사항
3. 정확한 파일 경로를 포함한 상위 3가지 조치 사항
4. 다음에 적용할 권장 ECC 스킬

## 체크리스트

- `hooks/hooks.json`, `scripts/hooks/`, 후크 테스트를 점검합니다.
- `skills/`, 명령어 커버리지, 에이전트 커버리지를 점검합니다.
- `.cursor/`, `.opencode/`, `.codex/` 간의 하네스 패리티(동등성)를 확인합니다.
- 깨지거나 오래된 참조에 플래그를 지정합니다.

## 결과 예시

```text
하네스 감사 (repo): 52/70
- 품질 관문: 9/10
- 평가 커버리지: 6/10
- 비용 효율성: 4/10

상위 3가지 조치 사항:
1) scripts/hooks/cost-tracker.js에 비용 추적 후크 추가
2) skills/eval-harness/SKILL.md에 pass@k 문서 및 템플릿 추가
3) .opencode/commands/에 /harness-audit 명령어 패리티 추가
```

## 인자 (Arguments)

$인자:
- `repo|hooks|skills|commands|agents` (선택적 대상 범위)
- `--format text|json` (선택적 출력 형식)
