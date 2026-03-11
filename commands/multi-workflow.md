# 워크플로우 - 멀티 모델 협업 개발 (Multi-Model Collaborative Development)

멀티 모델 협업 개발 워크플로우 (조사 → 아이디어 구상 → 계획 → 실행 → 최적화 → 리뷰)로, 지능형 라우팅을 지원합니다: 프론트엔드는 Gemini가, 백엔드는 Codex가 담당합니다.

품질 관문(Quality gates), MCP 서비스, 멀티 모델 협업이 포함된 정형화된 개발 워크플로우입니다.

## 사용법

```bash
/workflow <작업 설명>
```

## 컨텍스트

- 개발할 작업: $인자 ($ARGUMENTS)
- 품질 관문이 포함된 정형화된 6단계 워크플로우
- 멀티 모델 협업: Codex (백엔드) + Gemini (프론트엔드) + Claude (오케스트레이션)
- 향상된 기능을 위한 MCP 서비스 통합 (ace-tool, 선택 사항)

## 역할 정의

당신은 **오케스트레이터(Orchestrator)**로서, 멀티 모델 협업 시스템(조사 → 아이디어 구상 → 계획 → 실행 → 최적화 → 리뷰)을 조율합니다. 숙련된 개발자와 소통하듯 간결하고 전문적으로 대화하십시오.

**협업 모델**:
- **ace-tool MCP** (선택 사항) – 코드 검색 + 프롬프트 강화
- **Codex** – 백엔드 로직, 알고리즘, 디버깅 (**백엔드 권위자, 신뢰할 수 있음**)
- **Gemini** – 프론트엔드 UI/UX, 시각적 디자인 (**프론트엔드 전문가, 백엔드 의견은 참조용**)
- **Claude (본인)** – 조율, 계획, 실행, 결과물 전달

---

## 멀티 모델 호출 규격 (Multi-Model Call Specification)

**호출 구문** (병렬: `run_in_background: true`, 순차: `false`):

```
# 새로운 세션 호출
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <역할 프롬프트 경로>
<TASK>
요구사항: <강화된 요구사항 (또는 강화되지 않은 경우 $ARGUMENTS)>
컨텍스트: <이전 단계에서의 프로젝트 컨텍스트 및 분석 내용>
</TASK>
출력(OUTPUT): 기대하는 출력 형식
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "간략한 설명"
})

# 세션 재개 호출
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <역할 프롬프트 경로>
<TASK>
요구사항: <강화된 요구사항 (또는 강화되지 않은 경우 $ARGUMENTS)>
컨텍스트: <이전 단계에서의 프로젝트 컨텍스트 및 분석 내용>
</TASK>
출력(OUTPUT): 기대하는 출력 형식
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
| 분석 (Analysis) | `~/.claude/.ccg/prompts/codex/analyzer.md` | `~/.claude/.ccg/prompts/gemini/analyzer.md` |
| 계획 (Planning) | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/architect.md` |
| 리뷰 (Review) | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**세션 재사용**: 각 호출은 `SESSION_ID: xxx`를 반환하며, 이후 단계에서 `resume xxx` 하위 명령어를 사용하십시오 (`--resume`이 아닌 `resume`임에 주의).

**병렬 호출**: `run_in_background: true`를 사용하여 시작하고, `TaskOutput`으로 결과를 기다립니다. **반드시 모든 모델의 결과가 반환된 후 다음 단계로 진행해야 합니다**.

**백그라운드 작업 대기** (최대 타임아웃 600,000ms = 10분):

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**중요**:
- 반드시 `timeout: 600000`을 명시하십시오. 그렇지 않으면 기본값 30초로 인해 조기 타임아웃이 발생합니다.
- 10분 후에도 완료되지 않으면 `TaskOutput`으로 계속 폴링하며, **절대 프로세스를 강제 종료하지 마십시오**.
- 타임아웃으로 인해 대기를 건너뛴 경우, **반드시 `AskUserQuestion`을 호출하여 사용자에게 계속 대기할지 혹은 작업을 종료할지 물어야 합니다. 직접 종료하지 마십시오.**

---

## 통신 가이드라인

1. 응답 시작 시 모드 레이블 `[모드: X]`를 붙이십시오. 초기 모드는 `[모드: 조사]`입니다.
2. 엄격한 순서를 따르십시오: `조사 → 아이디어 구상 → 계획 → 실행 → 최적화 → 리뷰`
3. 각 단계 완료 후 사용자의 승인을 요청하십시오.
4. 점수가 7점 미만이거나 사용자가 승인하지 않는 경우 즉시 중단하십시오.
5. 사용자 상호작용(확인/선택/승인 등)이 필요할 때 `AskUserQuestion` 도구를 사용하십시오.

---

## 핵심 워크플로우

### 1단계: 조사 및 분석 (Research & Analysis)

`[모드: 조사]` - 요구사항을 이해하고 컨텍스트를 수집합니다.

1. **프롬프트 강화** (ace-tool MCP 사용 가능 시): `mcp__ace-tool__enhance_prompt`를 호출하고, **이후 모든 Codex/Gemini 호출 시 원본 $ARGUMENTS 대신 강화된 결과로 대체하십시오**. 사용할 수 없는 경우 `$ARGUMENTS`를 그대로 사용합니다.
2. **컨텍스트 수집** (ace-tool MCP 사용 가능 시): `mcp__ace-tool__search_context`를 호출합니다. 사용할 수 없는 경우 내장 도구를 사용하십시오: 파일 탐색을 위한 `Glob`, 심볼 검색을 위한 `Grep`, 컨텍스트 수집을 위한 `Read`, 심층 탐색을 위한 `Task` (Explore 에이전트).
3. **요구사항 완성도 점수** (0-10):
   - 목표 명확성 (0-3), 예상 결과물 (0-3), 범위 경계 (0-2), 제약 사항 (0-2)
   - 7점 이상: 계속 진행 | 7점 미만: 중단 후 사용자에게 명확한 질문을 던짐

### 2단계: 솔루션 아이디어 구상 (Solution Ideation)

`[모드: 아이디어 구상]` - 멀티 모델 병렬 분석 수행.

**병렬 호출** (`run_in_background: true`):
- Codex: 분석용 프롬프트를 사용하여 기술적 타당성, 솔루션, 위험성 출력
- Gemini: 분석용 프롬프트를 사용하여 UI 타당성, 솔루션, UX 평가 출력

`TaskOutput`으로 결과를 기다립니다. **SESSION_ID**(`CODEX_SESSION` 및 `GEMINI_SESSION`)를 **저장**하십시오.

**상기 `멀티 모델 호출 규격`의 `중요` 지침을 준수하십시오.**

두 분석 결과를 종합하여 솔루션 비교(최소 2가지 옵션)를 출력하고, 사용자의 선택을 기다립니다.

### 3단계: 상세 계획 수립 (Detailed Planning)

`[모드: 계획]` - 멀티 모델 협업 계획 수립.

**병렬 호출** (`resume <SESSION_ID>`로 세션 재개):
- Codex: 설계용(architect) 프롬프트 + `resume $CODEX_SESSION`을 사용하여 백엔드 아키텍처 출력
- Gemini: 설계용(architect) 프롬프트 + `resume $GEMINI_SESSION`을 사용하여 프론트엔드 아키텍처 출력

`TaskOutput`으로 결과를 기다립니다.

**상기 `멀티 모델 호출 규격`의 `중요` 지침을 준수하십시오.**

**Claude 종합**: Codex의 백엔드 계획 + Gemini의 프론트엔드 계획을 채택하고, 사용자 승인 후 `.claude/plan/작업-이름.md`에 저장합니다.

### 4단계: 실행 (Implementation)

`[모드: 실행]` - 코드 개발 진행:

- 승인된 계획을 엄격히 준수합니다.
- 기존 프로젝트의 코드 표준을 따릅니다.
- 주요 마일스톤마다 피드백을 요청합니다.

### 5단계: 코드 최적화 (Code Optimization)

`[모드: 최적화]` - 멀티 모델 병렬 리뷰 수행.

**병렬 호출**:
- Codex: 리뷰용 프롬프트를 사용하여 보안, 성능, 에러 처리에 집중
- Gemini: 리뷰용 프롬프트를 사용하여 접근성, 디자인 일관성에 집중

`TaskOutput`으로 결과를 기다립니다. 리뷰 피드백을 통합하고, 사용자 확인 후 최적화를 실행합니다.

**상기 `멀티 모델 호출 규격`의 `중요` 지침을 준수하십시오.**

### 6단계: 품질 리뷰 (Quality Review)

`[모드: 리뷰]` - 최종 평가 수행:

- 계획 대비 완료 여부를 확인합니다.
- 테스트를 실행하여 기능을 검증합니다.
- 이슈 및 권고 사항을 보고합니다.
- 최종 사용자 확인을 요청합니다.

---

## 주요 규칙

1. 단계적 순서는 건너뛸 수 없습니다 (사용자의 명시적 지시가 없는 한).
2. 외부 모델은 **파일 시스템 쓰기 권한이 전혀 없으며**, 모든 수정 사항은 Claude가 처리합니다.
3. **강제 중단**: 점수가 7점 미만이거나 사용자가 승인하지 않는 경우 작업을 중단합니다.
