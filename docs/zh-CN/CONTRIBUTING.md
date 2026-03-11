# Everything Claude Code 기여 가이드

기여해 주셔서 감사합니다! 이 저장소는 모든 Claude Code 사용자를 위한 커뮤니티 리소스입니다.

## 목차

* [모집 분야](#모집-분야)
* [빠른 시작](#빠른-시작)
* [스킬 기여하기](#스킬-기여하기)
* [에이전트 기여하기](#에이전트-기여하기)
* [후크 기여하기](#후크-기여하기)
* [명령어 기여하기](#명령어-기여하기)
* [풀 리퀘스트(PR) 프로세스](#풀-리퀘스트-프로세스)

***

## 모집 분야

### 에이전트

특정 작업을 잘 처리하는 새로운 에이전트:

* 언어별 리뷰어 (Python, Go, Rust)
* 프레임워크 전문가 (Django, Rails, Laravel, Spring)
* DevOps 전문가 (Kubernetes, Terraform, CI/CD)
* 도메인 전문가 (ML 파이프라인, 데이터 엔지니어링, 모바일)

### 스킬

워크플로우 정의 및 도메인 지식:

* 언어별 최선 관행(Best Practices)
* 프레임워크 패턴
* 테스트 전략
* 아키텍처 가이드라인

### 후크 (Hooks)

유용한 자동화:

* Lint/포맷팅 후크
* 보안 점검
* 검증 후크
* 알림 후크

### 명령어 (Commands)

유용한 워크플로우를 호출하는 슬래시 명령어:

* 배포 명령어
* 테스트 명령어
* 코드 생성 명령어

***

## 빠른 시작

```bash
# 1. 포크 및 클론
gh repo fork affaan-m/everything-claude-code --clone
cd everything-claude-code

# 2. 브랜치 생성
git checkout -b feat/my-contribution

# 3. 기여 내용 추가 (아래 섹션 참조)

# 4. 로컬에서 테스트
cp -r skills/my-skill ~/.claude/skills/  # 스킬의 경우
# 그 후 Claude Code로 테스트

# 5. PR 제출
git add . && git commit -m "feat: add my-skill" && git push -u origin feat/my-contribution
```

***

## 스킬 기여하기

스킬은 Claude Code가 컨텍스트에 따라 로드하는 지식 모듈입니다.

### 디렉토리 구조

```
skills/
└── your-skill-name/
    └── SKILL.md
```

### SKILL.md 템플릿

````markdown
---
name: your-skill-name
description: 스킬 목록에서 보여줄 짧은 설명
origin: ECC
---

# 스킬 제목

이 스킬이 다루는 내용에 대한 간략한 개요.

## 핵심 개념

주요 패턴과 가이드라인을 설명합니다.

## 코드 예시

```typescript
// 유용하고 검증된 예시를 포함하십시오.
function example() {
  // 주석이 잘 달린 코드
}
```
````

### 스킬 체크리스트

* [ ] 하나의 도메인/기술에 집중함
* [ ] 실용적인 코드 예시를 포함함
* [ ] 500라인 미만임
* [ ] 명확한 섹션 헤더를 사용함
* [ ] Claude Code로 테스트를 완료함

### 스킬 예시

| 스킬 | 목적 |
|-------|---------|
| `coding-standards/` | TypeScript/JavaScript 패턴 |
| `frontend-patterns/` | React 및 Next.js 최선 관행 |
| `backend-patterns/` | API 및 데이터베이스 패턴 |
| `security-review/` | 보안 체크리스트 |

***

## 에이전트 기여하기

에이전트는 작업을 위임받아 수행하는 전문 조수입니다.

### 파일 위치

```
agents/your-agent-name.md
```

### 에이전트 템플릿

```markdown
---
name: 에이전트 이름
description: 에이전트의 역할과 Claude가 언제 호출해야 하는지 구체적으로 작성하십시오.
tools: ["Read", "Write", "Edit", "Bash", "Grep", "Glob"]
model: sonnet
---

당신은 [역할] 전문가입니다.

## 당신의 역할

- 주요 책임
- 부차적 책임
- 하지 말아야 할 일 (경계 설정)

## 워크플로우

### 1단계: 이해
작업에 어떻게 접근하는지 설명하십시오.

### 2단계: 실행
작업을 어떻게 수행하는지 설명하십시오.

### 3단계: 검증
결과를 어떻게 검증하는지 설명하십시오.

## 출력 형식

사용자에게 반환하는 내용의 형식을 지정하십시오.

## 예시

### 예시: [시나리오]
입력: [사용자 제공 내용]
작업: [당신이 수행한 일]
출력: [반환한 내용]
```

### 에이전트 필드

| 필드 | 설명 | 옵션 |
|-------|-------------|---------|
| `name` | 소문자 및 하이픈 사용 | `code-reviewer` |
| `description` | 호출 시점을 결정하는 용도 | 매우 구체적으로 작성! |
| `tools` | 필요한 도구만 포함 | `Read, Write, Edit, Bash, Grep, Glob, WebFetch, Task` |
| `model` | 복잡도 수준 | `haiku` (단순), `sonnet` (코딩), `opus` (복잡) |

### 에이전트 예시

| 에이전트 | 목적 |
|-------|---------|
| `tdd-guide.md` | 테스트 주도 개발 |
| `code-reviewer.md` | 코드 리뷰 |
| `security-reviewer.md` | 보안 스캔 |
| `build-error-resolver.md` | 빌드 오류 수정 |

***

## 후크 기여하기

후크는 Claude Code 이벤트에 의해 트리거되는 자동화 동작입니다.

### 파일 위치

```
hooks/hooks.json
```

### 후크 유형

| 유형 | 트리거 시점 | 유스케이스 |
|------|---------|----------|
| `PreToolUse` | 도구 실행 전 | 검증, 경고, 차단 |
| `PostToolUse` | 도구 실행 후 | 포맷팅, 체크, 알림 |
| `SessionStart` | 세션 시작 시 | 컨텍스트 로드 |
| `Stop` | 세션 종료 시 | 정리, 감사 |

### 후크 형식

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "tool == \"Bash\" && tool_input.command matches \"rm -rf /\"",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[Hook] BLOCKED: Dangerous command' && exit 1"
          }
        ],
        "description": "위험한 rm 명령어 차단"
      }
    ]
  }
}
```

### 매처(Matcher) 구문

```javascript
// 특정 도구 매칭
tool == "Bash"
tool == "Edit"
tool == "Write"

// 입력 패턴 매칭
tool_input.command matches "npm install"
tool_input.file_path matches "\\.tsx?$"

// 조건 조합
tool == "Bash" && tool_input.command matches "git push"
```

### 후크 예시

```json
// tmux 외부에서 개발 서버 실행 차단
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"npm run dev\"",
  "hooks": [{"type": "command", "command": "echo '개발 서버는 tmux에서 실행하십시오' && exit 1"}],
  "description": "개발 서버가 tmux에서 실행되도록 보장"
}

// TypeScript 편집 후 자동 포맷팅
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.tsx?$\"",
  "hooks": [{"type": "command", "command": "npx prettier --write \"$file_path\""}],
  "description": "편집 후 TypeScript 파일 포맷팅"
}

// git push 전 경고
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"git push\"",
  "hooks": [{"type": "command", "command": "echo '[Hook] 푸시 전 변경 사항을 리뷰하십시오'"}],
  "description": "푸시 전 리뷰 알림"
}
```

### 후크 체크리스트

* [ ] 매처가 구체적임 (너무 광범위하지 않음)
* [ ] 명확한 오류/정보 메시지를 포함함
* [ ] 올바른 종료 코드 사용 (`exit 1` 차단, `exit 0` 허용)
* [ ] 충분히 테스트됨
* [ ] 설명이 포함됨

***

## 명령어 기여하기

명령어는 사용자가 `/명령어이름`으로 호출하는 작업입니다.

### 파일 위치

```
commands/your-command.md
```

### 명령어 템플릿

```markdown
---
description: /help에서 보여줄 짧은 설명
---

# 명령어 이름

## 목적

이 명령어가 하는 일.

## 사용법

`​`​`
/your-command [args]
`​`​`

## 워크플로우

1. 첫 번째 단계
2. 두 번째 단계
3. 마지막 단계

## 출력

사용자가 받게 될 결과물.
```

### 명령어 예시

| 명령어 | 목적 |
|---------|---------|
| `commit.md` | git 커밋 생성 |
| `code-review.md` | 코드 변경 사항 리뷰 |
| `tdd.md` | TDD 워크플로우 |
| `e2e.md` | E2E 테스트 |

***

## 풀 리퀘스트(PR) 프로세스

### 1. PR 제목 형식

```
feat(skills): add rust-patterns skill
feat(agents): add api-designer agent
feat(hooks): add auto-format hook
fix(skills): update React patterns
docs: improve contributing guide
```

### 2. PR 설명

```markdown
## 요약
무엇을 왜 추가하는지 작성하십시오.

## 유형
- [ ] 스킬
- [ ] 에이전트
- [ ] 후크
- [ ] 명령어

## 테스트
어떻게 테스트했는지 작성하십시오.

## 체크리스트
- [ ] 형식 가이드를 준수함
- [ ] Claude Code로 테스트 완료함
- [ ] 민감한 정보가 없음 (API 키, 경로 등)
- [ ] 설명이 명확함
```

### 3. 리뷰 프로세스

1. 메인테이너가 48시간 이내에 리뷰합니다.
2. 피드백이 있는 경우 이를 반영해 주십시오.
3. 승인되면 메인 브랜치에 병합됩니다.

***

## 가이드 원칙

### 권장 사항 (Dos)

* 기여 내용을 집중적이고 모듈화된 상태로 유지하십시오.
* 명확한 설명을 포함하십시오.
* 제출 전 반드시 테스트하십시오.
* 기존 패턴을 따르십시오.
* 의존성을 기록하십시오.

### 금지 사항 (Don'ts)

* 민감한 데이터(API 키, 토큰, 경로)를 포함하지 마십시오.
* 너무 복잡하거나 지엽적인 설정을 추가하지 마십시오.
* 테스트되지 않은 기여를 제출하지 마십시오.
* 기존 기능과 중복되는 내용을 만들지 마십시오.

***

## 파일 명명 규칙

* 소문자 및 하이픈 사용: `python-reviewer.md`
* 구체적으로 명명: `workflow.md` 대신 `tdd-workflow.md`
* 에이전트 이름과 파일 이름을 일치시킴

***

## 문의 사항

* **이슈:** [github.com/affaan-m/everything-claude-code/issues](https://github.com/affaan-m/everything-claude-code/issues)
* **X/Twitter:** [@affaanmustafa](https://x.com/affaanmustafa)

***

기여해 주셔서 감사합니다! 함께 훌륭한 리소스를 만들어 나갑시다.
