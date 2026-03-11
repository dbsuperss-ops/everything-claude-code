---
name: continuous-learning-v2
description: 후크(hook)를 통해 세션을 관찰하고, 확신 점수가 포함된 원자 단위의 '본능(instinct)'을 생성하며, 이를 스킬/명령/에이전트로 진화시키는 본능 기반 학습 시스템입니다. v2.1 버전에서는 프로젝트 간 간섭을 방지하기 위해 프로젝트 범위(scope)의 본능 기능이 추가되었습니다.
origin: ECC
version: 2.1.0
---

# 지속적 학습 v2.1 - 본능 기반 (Instinct-based)

확신 점수가 포함된 작은 학습된 행동 단위인 '본능(Instinct)'을 통해 Claude Code 세션을 재사용 가능한 지식으로 변환하는 고급 아키텍처 학습 시스템입니다.

**v2.1**에서는 **프로젝트 범위(Project-scoped) 본능**이 도입되었습니다. React 패턴은 React 프로젝트에, Python 규칙은 Python 프로젝트에 유지되며, "항상 데이터 검증 수행"과 같은 공통 패턴만 전역(Global)으로 공유됩니다.

## 적용 시점

* Claude Code 세션에서 자동 학습을 설정할 때
* 후크를 통한 본능 기반의 행동 추출을 구성할 때
* 학습된 행동의 확신 점수 임계값(Threshold)을 조정할 때
* 본능 라이브러리를 검토, 내보내기 또는 가져오기 할 때
* 본능을 완전한 스킬(Skill), 명령어(Command) 또는 에이전트(Agent)로 진화시킬 때
* 프로젝트 전용 본능과 전역 본능을 관리할 때
* 프로젝트 전용 본능을 전역 범위로 승격(Promote)시킬 때

## v2.1의 새로운 기능

| 기능 | v2.0 | v2.1 |
|---------|------|------|
| 저장 위치 | 전역 (~/.claude/homunculus/) | 프로젝트별 (projects/<hash>/) |
| 범위(Scope) | 모든 본능이 어디든 적용됨 | 프로젝트 전용 + 전역 분리 |
| 감지 방식 | 없음 | git remote URL / 저장소 경로 기반 |
| 승격(Promote) | 해당 없음 | 2개 이상의 프로젝트에서 발견 시 전역으로 승격 |
| 명령어 | 4개 (status/evolve/export/import) | 6개 (+promote/projects 추가) |
| 프로젝트 간 간섭 | 오염 위험 있음 | 기본적으로 격리됨 |

## v2와 v1의 차이점

| 기능 | v1 (기존) | v2 (현재) |
|---------|----|----|
| 관찰 방식 | 중단 훅 (세션 종료 시) | Pre/PostToolUse (100% 신뢰도) |
| 분석 주체 | 메인 컨텍스트 | 백그라운드 에이전트 (Haiku) |
| 분석 단위 | 완성된 형태의 스킬 | 원자 단위의 '본능' |
| 가중치 | 없음 | 0.3~0.9 확신 점수 부여 |
| 진화 과정 | 즉시 스킬로 생성 | 본능 -> 군집(Cluster) -> 스킬/명령/에이전트 |
| 공유 가능 | 없음 | 본능 내보내기/가져오기 가능 |

## 본능(Instinct) 모델

본능은 다음과 같은 형태의 작은 학습된 행동 단위입니다:

```yaml
---
id: prefer-functional-style
trigger: "새로운 함수를 작성할 때"
confidence: 0.7
domain: "code-style"
source: "session-observation"
scope: project
project_id: "a1b2c3d4e5f6"
project_name: "my-react-app"
---

# 함수형 스타일 선호

## 행동 (Action)
적절한 경우 클래스 대신 함수형 패턴을 사용합니다.

## 근거 (Evidence)
- 함수형 패턴 선호 사례 5회 관찰됨
- 2025-01-15 사용자가 클래스 기반 접근 방식을 함수형으로 수정함
```

**속성:**
* **원자성(Atomic)** -- 하나의 트리거, 하나의 행동
* **확신 점수 가중치** -- 0.3 = 시도 중, 0.9 = 거의 확실함
* **도메인 태그** -- 코드 스타일, 테스트, git, 디버깅, 워크플로우 등
* **근거 기반(Evidence-backed)** -- 어떤 관찰을 통해 생성되었는지 추적 가능
* **범위 인식(Scope-aware)** -- `project` (기본값) 또는 `global`

## 작동 원리

```
세션 활동 (git 저장소 내)
      |
      | 후크가 프롬프트 + 도구 사용 캡처 (100% 신뢰도)
      | + 프로젝트 컨텍스트 감지 (git remote / 저장소 경로)
      v
+---------------------------------------------+
|  projects/<project-hash>/observations.jsonl  |
|   (프롬프트, 도구 호출, 결과, 프로젝트 정보)   |
+---------------------------------------------+
      |
      | 관찰자 에이전트가 데이터 읽기 (백그라운드, Haiku)
      v
+---------------------------------------------+
|          패턴 탐지 (PATTERN DETECTION)       |
|   * 사용자 교정 -> 본능 생성                 |
|   * 에러 해결 -> 본능 생성                   |
|   * 반복되는 워크플로우 -> 본능 생성           |
|   * 범위 결정: 프로젝트 전용 또는 전역?        |
+---------------------------------------------+
      |
      | 생성 및 업데이트
      v
+---------------------------------------------+
|  projects/<project-hash>/instincts/personal/ |
|   * prefer-functional.yaml (0.7) [project]   |
|   * use-react-hooks.yaml (0.9) [project]     |
+---------------------------------------------+
|  instincts/personal/  (전역 범위)             |
|   * always-validate-input.yaml (0.85) [global]|
|   * grep-before-edit.yaml (0.6) [global]     |
+---------------------------------------------+
      |
      | /evolve (군집화) + /promote (승격)
      v
+---------------------------------------------+
|  projects/<hash>/evolved/ (프로젝트 범위)    |
|  evolved/ (전역 범위)                        |
|   * commands/new-feature.md                  |
|   * skills/testing-workflow.md               |
|   * agents/refactor-specialist.md            |
+---------------------------------------------+
```

## 프로젝트 감지

시스템은 현재 작업 중인 프로젝트를 자동으로 감지합니다:

1. **`CLAUDE_PROJECT_DIR` 환경 변수** (최우선순위)
2. **`git remote get-url origin`** -- 포터블한 프로젝트 ID 생성을 위해 해싱함 (다른 기기에서도 동일 저장소면 같은 ID 획득)
3. **`git rev-parse --show-toplevel`** -- 저장소 경로를 차선책으로 사용 (기기 종속적임)
4. **전역 폴백(Fallback)** -- 프로젝트가 감지되지 않으면 본능은 전역 범위로 저장됩니다.

각 프로젝트는 12자리의 해시 ID(예: `a1b2c3d4e5f6`)를 할당받습니다. `~/.claude/homunculus/projects.json` 파일이 이 ID와 사람이 읽을 수 있는 프로젝트 이름을 매핑합니다.

## 빠른 시작

### 1. 관찰 후크(Observation Hooks) 활성화

`~/.claude/settings.json`에 다음 내용을 추가합니다.

**플러그인으로 설치한 경우** (권장):

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }]
  }
}
```

**`~/.claude/skills`에 수동으로 설치한 경우**:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }]
  }
}
```

### 2. 디렉토리 구조 초기화

시스템이 처음 실행될 때 자동으로 생성하지만, 수동으로 생성할 수도 있습니다:

```bash
# 전역 디렉토리
mkdir -p ~/.claude/homunculus/{instincts/{personal,inherited},evolved/{agents,skills,commands},projects}

# 프로젝트 디렉토리는 git 저장소에서 후크가 처음 실행될 때 자동 생성됩니다.
```

### 3. 본능 명령어 사용

```bash
/instinct-status     # 학습된 본능 표시 (프로젝트 + 전역)
/evolve              # 관련된 본능들을 모아 스킬/명령어로 진화
/instinct-export     # 본능을 파일로 내보내기
/instinct-import     # 외부에서 본능 가져오기
/promote             # 프로젝트 본능을 전역 범위로 승격
/projects            # 알려진 모든 프로젝트와 본능 개수 목록 표시
```

## 명령어 요약

| 명령어 | 설명 |
|---------|-------------|
| `/instinct-status` | 모든 본능(프로젝트 + 전역)과 각 확신 점수 표시 |
| `/evolve` | 관련된 본능들을 스킬/명령어로 군집화하고 승격 제안 |
| `/instinct-export` | 본능 내보내기 (범위/도메인 필터링 가능) |
| `/instinct-import <file>` | 본능 가져오기 (범위 제어 가능) |
| `/promote [id]` | 특정 프로젝트 본능을 전역 범위로 승격 |
| `/projects` | 알려진 모든 프로젝트와 각각의 본능 개수 출력 |

## 설정

백그라운드 관찰자 동작을 제어하려면 `config.json`을 편집하십시오:

```json
{
  "version": "2.1",
  "observer": {
    "enabled": false,
    "run_interval_minutes": 5,
    "min_observations_to_analyze": 20
  }
}
```

| 키 | 기본값 | 설명 |
|-----|---------|-------------|
| `observer.enabled` | `false` | 백그라운드 관찰자 에이전트 활성화 여부 |
| `observer.run_interval_minutes` | `5` | 관찰자가 데이터를 분석하는 주기(분) |
| `observer.min_observations_to_analyze` | `20` | 분석을 실행하기 위해 필요한 최소 데이터 개수 |

기타 동작(관찰 캡처, 본능 임계값, 프로젝트 범위 결정 등)은 `instinct-cli.py` 및 `observe.sh` 파일 내의 기본값을 통해 설정됩니다.

## 파일 구조

```
~/.claude/homunculus/
+-- identity.json           # 사용자 프로필, 기술 수준 정보
+-- projects.json           # 레지스트리: 프로젝트 해시 -> 이름/경로/리모트 주소
+-- observations.jsonl      # 전역 관찰 데이터 (폴백용)
+-- instincts/
|   +-- personal/           # 전역적으로 자동 학습된 본능
|   +-- inherited/          # 전역적으로 가져온 본능
+-- evolved/
|   +-- agents/             # 전역적으로 생성된 에이전트
|   +-- skills/             # 전역적으로 생성된 스킬
|   +-- commands/           # 전역적으로 생성된 명령어
+-- projects/
    +-- a1b2c3d4e5f6/       # 프로젝트 해시 (git 리모트 URL 기반)
    |   +-- observations.jsonl
    |   +-- observations.archive/
    |   +-- instincts/
    |   |   +-- personal/   # 프로젝트 전용 자동 학습 본능
    |   |   +-- inherited/  # 프로젝트 전용 가져온 본능
    |   +-- evolved/
    |       +-- skills/
    |       +-- commands/
    |       +-- agents/
    +-- f6e5d4c3b2a1/       # 또 다른 프로젝트
        +-- ...
```

## 범위 결정 가이드라인 (Scope Decision)

| 패턴 유형 | 권장 범위 | 예시 |
|-------------|-------|---------|
| 언어/프레임워크 규칙 | **프로젝트** | "React hooks 사용", "Django REST 패턴 준수" |
| 파일 구조 선호도 | **프로젝트** | "테스트는 `__tests__`/에 배치", "컴포넌트는 src/components/에 배치" |
| 코딩 스타일 | **프로젝트** | "함수형 스타일 사용", "데이터 클래스 선호" |
| 에러 처리 전략 | **프로젝트** | "에러 처리에 Result 타입 사용" |
| 보안 관행 | **전역** | "사용자 입력 검증", "SQL 새니타이징" |
| 공통 베스트 프랙티스 | **전역** | "테스트 먼저 작성", "항상 에러 처리 수행" |
| 도구 워크플로우 선호도 | **전역** | "수정 전 Grep 권장", "쓰기 전 읽기 권장" |
| Git 관행 | **전역** | "Conventional Commit 준수", "작고 집중된 커밋" |

## 본능 승격 (프로젝트 -> 전역)

동일한 본능이 여러 프로젝트에서 높은 확신 점수로 반복되어 나타나면 전역 범위로 승결될 자격을 얻습니다.

**자동 승격 기준:**
* 동일한 본능 ID가 2개 이상의 프로젝트에서 발견됨
* 평균 확신 점수가 0.8 이상

**승격 방법:**
```bash
# 특정 본능 개별 승격
python3 instinct-cli.py promote prefer-explicit-errors

# 자격을 갖춘 모든 본능 자동 승격
python3 instinct-cli.py promote

# 실제 변경 없이 미리 보기 (Dry-run)
python3 instinct-cli.py promote --dry-run
```
`/evolve` 명령어 실행 시에도 승격 가능한 후보 본능을 제안해 줍니다.

## 확신 점수 (Confidence Scoring)

확신 점수는 시간이 지남에 따라 변합니다:

| 점수 | 의미 | 동작 방식 |
|-------|---------|----------|
| 0.3 | 시도 중 (Tentative) | 제안은 하되 강제하지 않음 |
| 0.5 | 중간 수준 (Moderate) | 관련 상황에서 적용 시도 |
| 0.7 | 강력함 (Strong) | 자동으로 적용 승인 |
| 0.9 | 거의 확실함 (Certain) | 핵심 행동 원칙으로 간주 |

**점수가 올라가는 경우:**
* 패턴이 반복적으로 관찰됨
* 사용자가 제안된 행동을 수정 없이 수긍함
* 다른 소스에서 유입된 유사한 본능과 일치함

**점수가 내려가는 경우:**
* 사용자가 해당 행동을 명시적으로 교정함
* 오랫동안 해당 패턴이 관찰되지 않음
* 상충하는 증거가 발견됨

## 왜 스킬 대신 후크를 사용하나요?

> "v1은 관찰을 위해 '스킬' 기능에 의존합니다. 스킬은 확률적입니다. Claude의 판단에 따라 약 50~80% 정도만 트리거됩니다."

후크는 **100% 트리거**되며 결정론적(deterministic)입니다. 이는 다음과 같은 이점이 있습니다:
* 모든 도구 호출이 누락 없이 기록됨
* 어떤 패턴도 놓치지 않음
* 학습 데이터가 종합적이고 체계적임

## 하위 호환성

v2.1은 v2.0 및 v1과 완전한 호환성을 유지합니다:
* `~/.claude/homunculus/instincts/` 경로의 기존 전역 본능은 계속 작동합니다.
* v1에서 생성된 `~/.claude/skills/learned/` 스킬들도 유효합니다.
* 중단 훅(Stop hook)은 계속 실행되지만, 이제는 v2 데이터로도 활용됩니다.
* 점진적 마이그레이션이 가능합니다 (두 방식 병렬 실행 가능).

## 개인정보 보호

* 관찰 데이터는 오직 사용자의 기기 **로컬**에만 저장됩니다.
* 프로젝트별 본능은 프로젝트 단위로 철저히 격리됩니다.
* 오직 **본능(패턴)**만 내보낼 수 있으며, 원본 관찰 데이터는 내보내지 않습니다.
* 실제 코드 내용이나 대화 전문은 공유되지 않습니다.
* 내보내거나 승격할 항목은 사용자가 직접 제어합니다.

## 관련 링크

* [Skill Creator](https://skill-creator.app) - 저장소 히스토리를 통한 본능 생성기
* Homunculus - 본능 기반 아키텍처(원자적 관찰, 확신 점수, 본능 진화 파이프라인)의 영감을 준 커뮤니티 프로젝트
* [Guide Full Version](https://x.com/affaanmustafa/status/2014040193557471352) - 지속적 학습 섹션

***

*본능 기반 학습: 하나의 프로젝트에서 배운 것을 넘어, Claude에게 당신만의 패턴을 가르치십시오.*
