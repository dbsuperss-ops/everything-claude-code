---
이름: evolve
설명: 본능(Instincts)을 분석하고 진화된 구조를 제안하거나 생성합니다.
명령어 여부: true
---

# Evolve 명령어

## 구현 방법

플러그인 루트 경로를 사용하여 instinct CLI를 실행합니다:

```bash
python3 "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/scripts/instinct-cli.py" evolve [--generate]
```

`CLAUDE_PLUGIN_ROOT`가 설정되지 않은 경우 (수동 설치 시):

```bash
python3 ~/.claude/skills/continuous-learning-v2/scripts/instinct-cli.py evolve [--generate]
```

본능(Instincts)을 분석하고 서로 관련된 것들을 고차원 구조로 클러스터링합니다:
- **명령어 (Commands)**: 본능이 사용자에 의해 호출되는 동작을 설명할 때
- **스킬 (Skills)**: 본능이 자동으로 트리거되는 동작을 설명할 때
- **에이전트 (Agents)**: 본능이 복잡하고 여러 단계로 이루어진 프로세스를 설명할 때

## 사용법

```
/evolve                    # 모든 본능을 분석하고 진화된 형태를 제안
/evolve --generate         # evolved/{skills,commands,agents} 아래에 파일도 생성
```

## 진화 규칙 (Evolution Rules)

### → 명령어 (사용자 호출형)
본능이 사용자가 명시적으로 요청할 동작을 설명할 때:
- "사용자가 ...를 요청할 때"에 대한 다수의 본능
- "새로운 X를 생성할 때"와 같은 트리거를 가진 본능
- 반복 가능한 시퀀스를 따르는 본능

예시:
- `new-table-step1`: "데이터베이스 테이블을 추가할 때, 마이그레이션을 생성한다"
- `new-table-step2`: "데이터베이스 테이블을 추가할 때, 스키마를 업데이트한다"
- `new-table-step3`: "데이터베이스 테이블을 추가할 때, 타입을 재생성한다"

→ 결과: **new-table** 명령어 생성

### → 스킬 (자동 트리거형)
본능이 자동으로 발생해야 하는 동작을 설명할 때:
- 패턴 매칭 트리거
- 에러 처리 응답
- 코드 스타일 강제

예시:
- `prefer-functional`: "함수를 작성할 때, 함수형 스타일을 선호한다"
- `use-immutable`: "상태를 수정할 때, 불변 패턴을 사용한다"
- `avoid-classes`: "모듈을 설계할 때, 클래스 기반 설계를 지양한다"

→ 결과: `functional-patterns` 스킬 생성

### → 에이전트 (심화/격리 필요형)
본능이 격리를 통해 이득을 얻는 복잡한 다단계 프로세스를 설명할 때:
- 디버깅 워크플로우
- 리팩토링 시퀀스
- 조사/리서치 작업

예시:
- `debug-step1`: "디버깅할 때, 먼저 로그를 확인한다"
- `debug-step2`: "디버깅할 때, 실패하는 컴포넌트를 격리한다"
- `debug-step3`: "디버깅할 때, 최소 재현 케이스를 만든다"
- `debug-step4`: "디버깅할 때, 테스트를 통해 수정을 검증한다"

→ 결과: **debugger** 에이전트 생성

## 처리 단계

1. 현재 프로젝트 컨텍스트 감지
2. 프로젝트 및 글로벌 본능 읽기 (ID 충돌 시 프로젝트 본능 우선)
3. 트리거/도메인 패턴별로 본능 그룹화
4. 다음 사항 식별:
   - 스킬 후보 (2개 이상의 본능을 가진 트리거 클러스터)
   - 명령어 후보 (높은 신뢰도의 워크플로우 본능)
   - 에이전트 후보 (더 크고 신뢰도가 높은 클러스터)
5. 가능한 경우 승격 후보(프로젝트 -> 글로벌) 표시
6. `--generate` 인자가 전달되면 다음 경로에 파일 작성:
   - 프로젝트 범위: `~/.claude/homunculus/projects/<project-id>/evolved/`
   - 글로벌 폴백: `~/.claude/homunculus/evolved/`

## 출력 형식 예시

```
============================================================
  진화 분석 (EVOLVE ANALYSIS) - 12개 본능
  프로젝트: my-app (a1b2c3d4e5f6)
  프로젝트 범위: 8개 | 글로벌 범위: 4개
============================================================

높은 신뢰도 본능 (>=80%): 5개

## 스킬 후보 (SKILL CANDIDATES)
1. 클러스터: "테스트 추가 (adding tests)"
   본능 수: 3개
   평균 신뢰도: 82%
   도메인: testing
   범위: project

## 명령어 후보 (COMMAND CANDIDATES) (2)
  /adding-tests
    출처: test-first-workflow [project]
    신뢰도: 84%

## 에이전트 후보 (AGENT CANDIDATES) (1)
  adding-tests-agent
    3개 본능 포함
    평균 신뢰도: 82%
```

## 플래그

- `--generate`: 분석 출력 외에 진화된 파일들을 실제로 생성

## 생성된 파일 형식

### 명령어 (Command)
```markdown
---
name: new-table
description: 마이그레이션 생성, 스키마 업데이트, 타입 생성을 포함하여 새로운 DB 테이블 생성
command: /new-table
evolved_from:
  - new-table-migration
  - update-schema
  - regenerate-types
---

# New Table 명령어

[클러스터링된 본능에 기반한 생성된 내용]

## 단계
1. ...
2. ...
```

### 스킬 (Skill)
```markdown
---
name: functional-patterns
description: 함수형 프로그래밍 패턴 강제
evolved_from:
  - prefer-functional
  - use-immutable
  - avoid-classes
---

# Functional Patterns 스킬

[클러스터링된 본능에 기반한 생성된 내용]
```

### 에이전트 (Agent)
```markdown
---
name: debugger
description: 체계적인 디버깅 에이전트
model: sonnet
evolved_from:
  - debug-check-logs
  - debug-isolate
  - debug-reproduce
---

# Debugger 에이전트

[클러스터링된 본능에 기반한 생성된 내용]
```
