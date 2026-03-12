# 명령어 → 에이전트 / 스킬 매핑 (Command → Agent / Skill Map)

이 문서는 각 슬래시 명령어와 그 명령어가 호출하는 주요 에이전트 또는 스킬을 나열합니다. 어떤 명령어가 어떤 에이전트를 사용하는지 확인하고, 리팩토링 시 일관성을 유지하기 위해 이 문서를 활용하십시오.

| 명령어 | 주요 에이전트 | 참고 사항 |
|---------|------------------|--------|
| `/plan` | planner | 코드 작성 전 구현 계획 수립 |
| `/tdd` | tdd-guide | 테스트 주도 개발(TDD) |
| `/code-review` | code-reviewer | 품질 및 보안 리뷰 |
| `/build-fix` | build-error-resolver | 빌드/타입 에러 수정 |
| `/e2e` | e2e-runner | Playwright E2E 테스트 |
| `/refactor-clean` | refactor-cleaner | 데드 코드(사용되지 않는 코드) 제거 |
| `/update-docs` | doc-updater | 문서 동기화 |
| `/update-codemaps` | doc-updater | 코드맵 / 아키텍처 문서 업데이트 |
| `/go-review` | go-reviewer | Go 코드 리뷰 |
| `/go-test` | tdd-guide | Go TDD 워크플로우 |
| `/go-build` | go-build-resolver | Go 빌드 에러 수정 |
| `/python-review` | python-reviewer | Python 코드 리뷰 |
| `/harness-audit` | — | 하네스 스코어카드 (단일 에이전트 없음) |
| `/loop-start` | loop-operator | 자율 루프 시작 |
| `/loop-status` | loop-operator | 루프 상태 점검 |
| `/quality-gate` | — | 품질 파이프라인 (후크 유사) |
| `/model-route` | — | 모델 추천 (에이전트 없음) |
| `/orchestrate` | planner, tdd-guide, code-reviewer, security-reviewer, architect | 멀티 에이전트 핸드오프(handoff) |
| `/multi-plan` | architect (Codex/Gemini 프롬프트) | 멀티 모델 계획 수립 |
| `/multi-execute` | architect / frontend 프롬프트 | 멀티 모델 실행 |
| `/multi-backend` | architect | 백엔드 멀티 서비스 |
| `/multi-frontend` | architect | 프론트엔드 멀티 서비스 |
| `/multi-workflow` | architect | 일반 멀티 서비스 |
| `/learn` | — | 지속적 학습(continuous-learning) 스킬, 인스팅트 |
| `/learn-eval` | — | 지속적 학습 v2, 평가 후 저장 |
| `/instinct-status` | — | 지속적 학습 v2 |
| `/instinct-import` | — | 지속적 학습 v2 |
| `/instinct-export` | — | 지속적 학습 v2 |
| `/evolve` | — | 지속적 학습 v2, 인스팅트 클러스터링 |
| `/promote` | — | 지속적 학습 v2 |
| `/projects` | — | 지속적 학습 v2 |
| `/skill-create` | — | skill-create-output 스크립트, git 이력 활용 |
| `/checkpoint` | — | 검증 루프(verification-loop) 스킬 |
| `/verify` | — | 검증 루프 스킬 |
| `/eval` | — | 에발 하네스(eval-harness) 스킬 |
| `/test-coverage` | — | 커버리지 분석 |
| `/sessions` | — | 세션 이력 관리 |
| `/setup-pm` | — | 패키지 매니저 설정 스크립트 |
| `/claw` | — | NanoClaw CLI (scripts/claw.js) |
| `/pm2` | — | PM2 서비스 라이프사이클 관리 |
| `/security-scan` | security-reviewer (스킬) | security-scan 스킬을 통한 AgentShield 실행 |

## 명령어에서 참조하는 스킬들 (Skills referenced by commands)

- **지속적 학습(continuous-learning)**, **지속적 학습 v2**: `/learn`, `/learn-eval`, `/instinct-*`, `/evolve`, `/promote`, `/projects`
- **검증 루프(verification-loop)**: `/checkpoint`, `/verify`
- **에발 하네스(eval-harness)**: `/eval`
- **보안 스캔(security-scan)**: `/security-scan` (AgentShield 실행)
- **전략적 압축(strategic-compact)**: 압축 지점(후크)에서 제안됨

## 이 지도를 활용하는 방법

- **탐색성(Discoverability):** 어떤 명령어가 어떤 에이전트를 트리거하는지 찾으십시오. (예: "code-reviewer를 사용하려면 `/code-review`를 사용")
- **리팩토링:** 에이전트의 이름을 변경하거나 제거할 때, 이 문서와 명령어 파일들을 검색하여 참조를 확인하십시오.
- **CI/문서화:** 카탈로그 스크립트(`node scripts/ci/catalog.js`)가 에이전트/명령어/스킬의 개수를 출력하면, 이 지도는 명령어와 에이전트 사이의 관계를 보완해 줍니다.
