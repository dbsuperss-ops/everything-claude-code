# 실행 - 멀티 모델 협업 실행 (Multi-Model Collaborative Execution)

멀티 모델 협업 실행 — 계획(Plan)에서 프로토타입 획득 → Claude가 리팩토링 및 구현 → 멀티 모델 감사 및 결과물 전달.

$인자 (ARGUMENTS)

---

## 핵심 프로토콜

- **언어 프로토콜**: 도구/모델과 상호작용할 때는 **영어(English)**를 사용하고, 사용자와 소통할 때는 사용자의 언어(한국어)를 사용합니다.
- **코드 주권**: 외부 모델은 **파일 시스템 쓰기 권한이 전혀 없으며**, 모든 수정 사항은 Claude가 처리합니다.
- **프로토타입 리팩토링**: Codex/Gemini의 Unified Diff 출력을 "기술 부채가 있는 프로토타입"으로 간주하고, 반드시 프로덕션 등급의 코드로 리팩토링해야 합니다.
- **손절매(Stop-Loss) 메커니즘**: 현재 단계의 출력이 검증되기 전까지 다음 단계로 진행하지 마십시오.
- **전제 조건**: 사용자가 `/ccg:plan` 출력에 대해 명시적으로 "Y"라고 답한 후에만 실행하십시오 (확인이 누락된 경우 먼저 확인해야 함).

---

## 멀티 모델 호출 규격 (Multi-Model Call Specification)

**호출 구문** (병렬: `run_in_background: true` 사용):

```
# 세션 재개 호출 (권장) - 구현 프로토토타입
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <역할 프롬프트 경로>
<TASK>
요구사항: <작업 설명>
컨텍스트: <계획 내용 + 대상 파일들>
</TASK>
출력(OUTPUT): Unified Diff Patch만 출력하십시오. 실제 파일 수정은 엄격히 금지됩니다.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "간략한 설명"
})

# 새로운 세션 호출 - 구현 프로토토타입
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <역할 프롬프트 경로>
<TASK>
요구사항: <작업 설명>
컨텍스트: <계획 내용 + 대상 파일들>
</TASK>
출력(OUTPUT): Unified Diff Patch만 출력하십시오. 실제 파일 수정은 엄격히 금지됩니다.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "간략한 설명"
})
```

**감사 호출 구문** (코드 리뷰 / 감사):

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <역할 프롬프트 경로>
<TASK>
범위: 최종 코드 변경 사항 감사.
입력:
- 적용된 패치 (git diff / 최종 unified diff)
- 수정된 파일들 (필요한 경우 관련 발췌본)
제약 사항:
- 어떠한 파일도 수정하지 마십시오.
- 파일 시스템 접근을 가정하는 도구 명령어를 출력하지 마십시오.
</TASK>
출력(OUTPUT):
1) 우선순위가 지정된 이슈 목록 (심각도, 파일, 근거)
2) 구체적인 수정 방안. 코드 변경이 필요한 경우 코드 블록 내에 Unified Diff Patch를 포함하십시오.
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "간략한 설명"
})
```

**모델 파라미터 참고**:
- `{{GEMINI_MODEL_FLAG}}`: `--backend gemini` 사용 시 `--gemini-model gemini-3-pro-preview `(뒤에 공백 포함)로 대체하십시오. codex의 경우 빈 문자열을 사용합니다.

**역할 프롬프트**:

| 단계 | Codex | Gemini |
|-------|-------|--------|
| 구현 (Implementation) | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/frontend.md` |
| 리뷰 (Review) | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**세션 재사용**: `/ccg:plan`에서 SESSION_ID를 제공한 경우, `resume <SESSION_ID>`를 사용하여 컨텍스트를 재사용하십시오.

**백그라운드 작업 대기** (최대 타임아웃 600,000ms = 10분):

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**중요**:
- 반드시 `timeout: 600000`을 명시하십시오. 그렇지 않으면 기본값 30초로 인해 조기 타임아웃이 발생합니다.
- 10분 후에도 완료되지 않으면 `TaskOutput`으로 계속 폴링(polling)하며, **절대 프로세스를 강제 종료하지 마십시오**.
- 타임아웃으로 인해 대기를 건너뛴 경우, **반드시 `AskUserQuestion`을 호출하여 사용자에게 대기를 계속할지 또는 작업을 종료할지 물어야 합니다**.

---

## 실행 워크플로우

**실행 작업**: $인자 ($ARGUMENTS)

### 0단계: 계획 읽기

`[모드: 준비]`

1. **입력 유형 식별**:
   - 계획 파일 경로 (예: `.claude/plan/xxx.md`)
   - 직접적인 작업 설명

2. **계획 내용 읽기**:
   - 계획 파일 경로가 제공된 경우 파일 내용을 읽고 파싱합니다.
   - 작업 유형, 구현 단계, 주요 파일, SESSION_ID를 추출합니다.

3. **실행 전 확인**:
   - 입력이 "직접적인 작업 설명"이거나 계획에 `SESSION_ID` / 주요 파일 정보가 누락된 경우 먼저 사용자에게 확인합니다.
   - 사용자가 계획에 "Y"라고 답했는지 확인할 수 없는 경우 진행 전 다시 확인해야 합니다.

4. **작업 유형 라우팅**:

   | 작업 유형 | 감지 기준 | 경로 |
   |-----------|-----------|-------|
   | **프론트엔드** | 페이지, 컴포넌트, UI, 스타일, 레이아웃 | Gemini |
   | **백엔드** | API, 인터페이스, 데이터베이스, 로직, 알고리즘 | Codex |
   | **풀스택** | 프론트엔드와 백엔드 모두 포함 | Codex ∥ Gemini 병렬 |

---

### 1단계: 빠른 컨텍스트 수집

`[모드: 검색]`

**ace-tool MCP를 사용할 수 있는 경우**, 이를 사용하여 컨텍스트를 빠르게 수집합니다:

계획의 "주요 파일(Key Files)" 목록을 기반으로 `mcp__ace-tool__search_context`를 호출합니다:

```
mcp__ace-tool__search_context({
  query: "<계획 내용, 주요 파일, 모듈, 함수 이름을 포함한 시맨틱 쿼리>",
  project_root_path: "$PWD"
})
```

**수집 전략**:
- 계획의 "주요 파일" 테이블에서 대상 경로를 추출합니다.
- 진입점 파일, 의존성 모듈, 관련 타입 정의를 아우르는 시맨틱 쿼리를 구성합니다.
- 결과가 불충분할 경우 1-2회 재귀적 검색을 수행합니다.

**ace-tool MCP를 사용할 수 없는 경우**, Claude Code 내장 도구를 폴백(fallback)으로 사용합니다:
1. **Glob**: 계획의 "주요 파일" 테이블에서 대상 파일을 찾습니다 (예: `Glob("src/components/**/*.tsx")`).
2. **Grep**: 코드베이스 전체에서 핵심 심볼, 함수 이름, 타입 정의를 검색합니다.
3. **Read**: 발견된 파일들을 읽어 전체 컨텍스트를 수집합니다.
4. **Task (Explore 에이전트)**: 더 광범위한 탐색이 필요한 경우 `subagent_type: "Explore"`와 함께 `Task`를 사용합니다.

**수집 완료 후**:
- 수집된 코드 스니펫을 정리합니다.
- 구현을 위한 완전한 컨텍스트가 확보되었는지 확인합니다.
- 3단계로 진행합니다.

---

### 3단계: 프로토타입 획득

`[모드: 프로토타입]`

**작업 유형에 따른 라우팅**:

#### 경로 A: 프론트엔드/UI/스타일 → Gemini

**제한**: 컨텍스트 < 32k 토큰

1. Gemini 호출 (`~/.claude/.ccg/prompts/gemini/frontend.md` 사용)
2. 입력: 계획 내용 + 수집된 컨텍스트 + 대상 파일들
3. 출력(OUTPUT): `Unified Diff Patch만 출력하십시오. 실제 파일 수정은 엄격히 금지됩니다.`
4. **Gemini는 프론트엔드 디자인 권위자이며, 그 CSS/React/Vue 프로토타입은 최종 시각적 기준선이 됩니다.**
5. **주의**: Gemini의 백엔드 로직 제안은 무시하십시오.
6. 계획에 `GEMINI_SESSION`이 있는 경우 `resume <GEMINI_SESSION>`을 선호합니다.

#### 경로 B: 백엔드/로직/알고리즘 → Codex

1. Codex 호출 (`~/.claude/.ccg/prompts/codex/architect.md` 사용)
2. 입력: 계획 내용 + 수집된 컨텍스트 + 대상 파일들
3. 출력(OUTPUT): `Unified Diff Patch만 출력하십시오. 실제 파일 수정은 엄격히 금지됩니다.`
4. **Codex는 백엔드 로직 권위자이며, 그 논리적 추론 및 디버깅 능력을 활용하십시오.**
5. 계획에 `CODEX_SESSION`이 있는 경우 `resume <CODEX_SESSION>`을 선호합니다.

#### 경로 C: 풀스택 → 병렬 호출

1. **병렬 호출** (`run_in_background: true`):
   - Gemini: 프론트엔드 파트 처리
   - Codex: 백엔드 파트 처리
2. `TaskOutput`을 사용하여 두 모델의 결과가 모두 완료될 때까지 기다립니다.
3. 각 모델은 계획에 명시된 `SESSION_ID`를 사용하여 `resume`합니다 (SESSION_ID가 없는 경우 새 세션 생성).

**상기 `멀티 모델 호출 규격` 섹션의 `중요` 지침을 준수하십시오.**

---

### 4단계: 코드 구현

`[모드: 구현]`

**코드 주권자(Code Sovereign)인 Claude가 다음 단계들을 실행합니다**:

1. **Diff 읽기**: Codex/Gemini가 반환한 Unified Diff Patch를 파싱합니다.

2. **사고 샌드박스 (Mental Sandbox)**:
   - 대상 파일들에 Diff를 적용하는 과정을 시뮬레이션합니다.
   - 논리적 일관성을 확인합니다.
   - 잠재적 충돌이나 부작용(side effects)을 식별합니다.

3. **리팩토링 및 정리**:
   - 가져온 프로토타입 코드를 **가독성이 높고 유지보수가 쉬운 엔터프라이즈 등급의 코드**로 리팩토링합니다.
   - 불필요한 코드를 제거합니다.
   - 프로젝트의 기존 코드 표준을 준수하도록 보장합니다.
   - **필요하지 않은 경우 주석이나 문서를 생성하지 마십시오.** 코드는 그 자체로 설명이 되어야 합니다.

4. **최소 범위 수정**:
   - 변경 사항을 요구사항 범위로 한정합니다.
   - 부작용에 대해 **반드시 검토**하십시오.
   - 타겟팅된 수정을 수행합니다.

5. **변경 사항 적용**:
   - Edit/Write 도구를 사용하여 실제 수정을 실행합니다.
   - **필요한 코드만 수정**하며, 사용자의 기존 기능을 건드리지 않도록 주의하십시오.

6. **자가 검증** (강력 권장):
   - 프로젝트의 기존 린트(lint) / 타입 체크 / 테스트를 실행합니다 (관련된 최소 범위 우선).
   - 실패 시: 회귀 결함(regression)을 먼저 수정한 후 5단계로 진행합니다.

---

### 5단계: 감사 및 전달

`[모드: 감사]`

#### 5.1 자동 감사

**변경 사항이 적용된 후, 즉시 Codex와 Gemini를 병렬 호출하여 코드 리뷰를 수행해야 합니다**:

1. **Codex 리뷰** (`run_in_background: true`):
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/reviewer.md`
   - 입력: 변경된 Diff + 대상 파일들
   - 초점: 보안, 성능, 에러 처리, 로직 정확성

2. **Gemini 리뷰** (`run_in_background: true`):
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/reviewer.md`
   - 입력: 변경된 Diff + 대상 파일들
   - 초점: 접근성, 디자인 일관성, 사용자 경험

`TaskOutput`을 사용하여 두 모델의 리뷰 결과가 완료될 때까지 기다립니다. 컨텍스트 일관성을 위해 3단계의 세션을 재사용(`resume <SESSION_ID>`)하는 것을 권장합니다.

#### 5.2 통합 및 수정

1. Codex + Gemini의 리뷰 피드백을 종합합니다.
2. 신뢰 규칙에 따라 가중치를 둡니다: 백엔드는 Codex를, 프론트엔드는 Gemini를 따릅니다.
3. 필요한 수정을 실행합니다.
4. 위험이 수용 가능한 수준이 될 때까지 필요에 따라 5.1단계를 반복합니다.

#### 5.3 전달 확인

감사를 통과한 후 사용자에게 보고합니다:

```markdown
## 실행 완료

### 변경 요약
| 파일 | 작업 | 설명 |
|------|-----------|-------------|
| path/to/file.ts | 수정됨 | 상세 설명 |

### 감사 결과
- Codex: <통과/N개의 이슈 발견>
- Gemini: <통과/N개의 이슈 발견>

### 권고 사항
1. [ ] <권장 테스트 단계>
2. [ ] <권장 검증 단계>
```

---

## 주요 규칙

1. **코드 주권** – 모든 파일 수정은 Claude가 수행하며, 외부 모델은 직접 수정 권한이 없습니다.
2. **프로토타입 리팩토링** – Codex/Gemini의 출력은 초안으로 간주하며, 반드시 리팩토링해야 합니다.
3. **신뢰 규칙** – 백엔드는 Codex를, 프론트엔드는 Gemini를 따릅니다.
4. **최소 변경** – 필요한 코드만 수정하며 부작용을 방지합니다.
5. **필수 감사** – 변경 후 반드시 멀티 모델 코드 리뷰를 수행해야 합니다.

---

## 사용법

```bash
# 계획 파일 실행
/ccg:execute .claude/plan/기능-이름.md

# 작업 직접 실행 (이미 컨텍스트에서 논의된 경우)
/ccg:execute 이전 계획을 바탕으로 사용자 인증 구현해줘
```

---

## /ccg:plan과의 관계

1. `/ccg:plan`이 계획과 SESSION_ID를 생성합니다.
2. 사용자가 "Y"로 확인합니다.
3. `/ccg:execute`가 계획을 읽고 SESSION_ID를 재사용하여 구현을 실행합니다.
