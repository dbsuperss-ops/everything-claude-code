# Everything Claude Code 기여 가이드 (Contributing)

기여해 주셔서 감사합니다! 이 레포지토리는 Claude Code 사용자를 위한 커뮤니티 리소스입니다.

## 목차

- [찾고 있는 기여 내용](#what-were-looking-for)
- [빠른 시작](#quick-start)
- [스킬 기여하기](#contributing-skills)
- [에이전트 기여하기](#contributing-agents)
- [후크 기여하기](#contributing-hooks)
- [명령어 기여하기](#contributing-commands)
- [풀 리퀘스트(PR) 프로세스](#pull-request-process)

---

## 찾고 있는 기여 내용

### 에이전트 (Agents)
특정 작업을 잘 처리하는 새로운 에이전트:
- 언어별 리뷰어 (Python, Go, Rust 등)
- 프레임워크 전문가 (Django, Rails, Laravel, Spring 등)
- DevOps 전문가 (Kubernetes, Terraform, CI/CD 등)
- 도메인 전문가 (ML 파이프라인, 데이터 엔지니어링, 모바일 등)

### 스킬 (Skills)
워크플로우 정의 및 도메인 지식:
- 언어별 최선 관행 (Best practices)
- 프레임워크 패턴
- 테스트 전략
- 아키텍처 가이드

### 후크 (Hooks)
유용한 자동화:
- 린팅(Linting)/포맷팅 후크
- 보안 점검
- 검증 후크
- 알림 후크

### 명령어 (Commands)
유용한 워크플로우를 호출하는 슬래시 명령어:
- 배포 명령어
- 테스트 명령어
- 코드 생성 명령어

---

## 빠른 시작

```bash
# 1. 포크 및 클론
gh repo fork affaan-m/everything-claude-code --clone
cd everything-claude-code

# 2. 브랜치 생성
git checkout -b feat/my-contribution

# 3. 기여 내용 추가 (아래 섹션 참조)

# 4. 로컬 테스트
cp -r skills/my-skill ~/.claude/skills/  # 스킬의 경우
# 그 다음 Claude Code로 테스트

# 5. PR 제출
git add . && git commit -m "feat: add my-skill" && git push -u origin feat/my-contribution
```

---

## 스킬 기여하기

스킬은 Claude Code가 컨텍스트에 따라 로드하는 지식 모듈입니다.

### 디렉토리 구조

```
skills/
└── your-skill-name/
    └── SKILL.md
```

### SKILL.md 템플릿

```markdown
---
name: your-skill-name
description: 스킬 목록에 표시될 간단한 설명
origin: ECC
---

# 스킬 제목

이 스킬이 다루는 내용에 대한 간략한 개요.

## 핵심 개념 (Core Concepts)

주요 패턴과 가이드라인을 설명합니다.

## 코드 예시 (Code Examples)

\`\`\`typescript
// 실용적이고 테스트된 예시를 포함하십시오
function example() {
  // 주석이 잘 달린 코드
}
\`\`\`

## 최선의 관행 (Best Practices)

- 실천 가능한 가이드라인
- 권장 사항(Do's) 및 금지 사항(Don'ts)
- 피해야 할 일반적인 함정

## 사용 시기

이 스킬이 적용되는 시나리오를 설명합니다.
```

### 스킬 체크리스트

- [ ] 하나의 도메인/기술에 집중되어 있는가
- [ ] 실용적인 코드 예시가 포함되어 있는가
- [ ] 500라인 이내인가
- [ ] 명확한 섹션 헤더를 사용했는가
- [ ] Claude Code로 테스트했는가

---

## 에이전트 기여하기

에이전트는 Task 도구를 통해 호출되는 전문 어시스턴트입니다.

### 파일 위치

```
agents/your-agent-name.md
```

### 에이전트 템플릿

```markdown
---
name: your-agent-name
description: 이 에이전트가 하는 일과 Claude가 언제 호출해야 하는지 기재. 구체적으로 작성하십시오!
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet
---

당신은 [역할] 전문가입니다.

## 역할

- 주요 책임
- 부차적 책임
- 하지 않는 일 (경계 설정)

## 워크플로우

### 1단계: 이해 (Understand)
과업에 접근하는 방식.

### 2단계: 실행 (Execute)
작업을 수행하는 방식.

### 3단계: 검증 (Verify)
결과를 확인하는 방식.

## 출력 형식

사용자에게 반환하는 내용.

## 예시

### 예시: [시나리오]
입력: [사용자가 제공하는 것]
동작: [당신이 하는 일]
출력: [당신이 반환하는 것]
```

### 에이전트 필드

| 필드 | 설명 | 옵션 |
|-------|-------------|---------|
| `name` | 하이픈으로 연결된 소문자 | `code-reviewer` |
| `description` | 호출 여부를 결정할 때 사용 | 구체적으로 작성! |
| `tools` | 필요한 도구만 선택 | `Read, Write, Edit, Bash, Grep, Glob, WebFetch, Task` |
| `model` | 복잡도 수준 | `haiku` (단순), `sonnet` (코딩), `opus` (복잡) |

---

## 후크 기여하기

후크는 Claude Code 이벤트에 의해 트리거되는 자동화된 동작입니다.

### 파일 위치

```
hooks/hooks.json
```

### 후크 유형

| 유형 | 트리거 | 사용 사례 |
|------|---------|----------|
| `PreToolUse` | 도구 실행 전 | 검증, 경고, 차단 |
| `PostToolUse` | 도구 실행 후 | 포맷팅, 확인, 알림 |
| `SessionStart` | 세션 시작 시 | 컨텍스트 로드 |
| `Stop` | 세션 종료 시 | 정리, 감사(Audit) |

---

## 명령어 기여하기

명령어는 사용자가 `/command-name`으로 호출하는 동작입니다.

### 파일 위치

```
commands/your-command.md
```

### 명령어 템플릿

```markdown
---
description: /help에 표시될 간단한 설명
---

# 명령어 이름

## 목적

이 명령어가 하는 일.

## 사용법

\`\`\`
/your-command [인자]
\`\`\`

## 워크플로우

1. 첫 번째 단계
2. 두 번째 단계
3. 최종 단계

## 출력

사용자가 받는 결과물.
```

---

## 풀 리퀘스트(PR) 프로세스

### 1. PR 제목 형식

```
feat(skills): add rust-patterns skill
feat(agents): add api-designer agent
feat(hooks): add auto-format hook
fix(skills): update React patterns
docs: improve contributing guide
```

### 2. PR 설명 (PR Description)

```markdown
## 요약 (Summary)
무엇을 왜 추가하는지 작성하십시오.

## 유형
- [ ] 스킬 (Skill)
- [ ] 에이전트 (Agent)
- [ ] 후크 (Hook)
- [ ] 명령어 (Command)

## 테스트
어떻게 테스트했는지 작성하십시오.

## 체크리스트
- [ ] 형식 가이드라인을 준수함
- [ ] Claude Code로 테스트함
- [ ] 민감한 정보(API 키, 경로 등) 없음
- [ ] 설명이 명확함
```

### 3. 리뷰 프로세스

1. 메인테이너가 48시간 이내에 검토합니다.
2. 요청이 있는 경우 피드백을 반영하십시오.
3. 승인되면 `main` 브랜치에 병합됩니다.

---

## 가이드라인 (Guidelines)

### 권장 사항 (Do)
- 기여 내용을 집중적이고 모듈화된 상태로 유지하십시오.
- 명확한 설명을 포함하십시오.
- 제출 전에 테스트하십시오.
- 기존 패턴을 따르십시오.
- 의존성을 문서화하십시오.

### 금지 사항 (Don't)
- 민감한 데이터(API 키, 토큰, 경로 등)를 포함하지 마십시오.
- 지나치게 복잡하거나 지엽적인 설정을 추가하지 마십시오.
- 테스트되지 않은 기여를 제출하지 마십시오.
- 기존 기능과 중복되는 것을 만들지 마십시오.

---

## 파일 명명 규칙

- 하이픈으로 연결된 소문자를 사용하십시오: `python-reviewer.md`
- 설명적인 이름을 사용하십시오: `workflow.md` 보다는 `tdd-workflow.md`
- 이름과 파일명을 일치시키십시오.

---

## 질문이 있으신가요?

- **이슈 (Issues):** [github.com/affaan-m/everything-claude-code/issues](https://github.com/affaan-m/everything-claude-code/issues)
- **X/Twitter:** [@affaanmustafa](https://x.com/affaanmustafa)

---

기여해 주셔서 감사합니다! 함께 훌륭한 리소스를 만들어 나갑시다.
    
