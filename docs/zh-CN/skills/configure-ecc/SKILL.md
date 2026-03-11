---
name: configure-ecc
description: Everything Claude Code의 대화형 설치 프로그램입니다. 사용자가 스킬과 규칙을 사용자 수준 또는 프로젝트 수준 디렉토리에 선택하여 설치하도록 안내하고, 경로를 검증하며, 선택적으로 설치된 파일을 최적화합니다.
origin: ECC
---

# Everything Claude Code (ECC) 구성

Everything Claude Code 프로젝트를 위한 단계별 대화형 설치 마법사입니다. `AskUserQuestion`을 사용하여 사용자가 스킬과 규칙을 선택적으로 설치하도록 안내하고, 설치의 정확성을 검증하며 최적화 작업을 제공합니다.

## 적용 시점

* 사용자가 "configure ecc", "install ecc", "setup everything claude code" 또는 이와 유사한 요청을 할 때
* 사용자가 본 프로젝트의 스킬이나 규칙을 선택적으로 설치하고 싶어 할 때
* 기존 ECC 설치 상태를 검증하거나 수정하고 싶어 할 때
* 프로젝트에 맞춰 설치된 스킬이나 규칙을 최적화하고 싶을 때

## 사전 요구 사항

이 스킬은 활성화되기 전에 Claude Code가 접근 가능해야 합니다. 다음 두 가지 방법 중 하나로 시작할 수 있습니다:

1. **플러그인을 통한 방법**: `/plugin install everything-claude-code` 실행 — 플러그인이 자동으로 이 스킬을 로드합니다.
2. **수동 방법**: 이 스킬 파일만 `~/.claude/skills/configure-ecc/SKILL.md` 경로에 복사한 후, "configure ecc"라고 말하여 활성화합니다.

***

## 0단계: ECC 저장소 클론

설치를 시작하기 전에 최신 ECC 소스 코드를 `/tmp` 디렉토리에 클론합니다:

```bash
rm -rf /tmp/everything-claude-code
git clone https://github.com/affaan-m/everything-claude-code.git /tmp/everything-claude-code
```

이후 모든 복사 작업의 원본 경로를 `ECC_ROOT=/tmp/everything-claude-code`로 설정합니다.

클론이 실패하는 경우(네트워크 문제 등), `AskUserQuestion`을 호출하여 사용자에게 기존에 클론된 ECC의 로컬 경로를 물어보십시오.

***

## 1단계: 설치 수준 선택

`AskUserQuestion`을 사용하여 설치 위치를 확인합니다:

```
질문: "ECC 구성 요소를 어디에 설치할까요?"
옵션:
  - "사용자 수준 (~/.claude/)" — "모든 Claude Code 프로젝트에 적용됩니다"
  - "프로젝트 수준 (.claude/)" — "현재 프로젝트에만 적용됩니다"
  - "둘 다" — "공통/공유 항목은 사용자 수준에, 프로젝트 전용 항목은 프로젝트 수준에 설치합니다"
```

선택 결과를 `INSTALL_LEVEL`에 저장하고 대상 디렉토리를 설정합니다:

* 사용자 수준: `TARGET=~/.claude`
* 프로젝트 수준: `TARGET=.claude` (현재 프로젝트 루트 기준)
* 둘 다: `TARGET_USER=~/.claude`, `TARGET_PROJECT=.claude`

대상 디렉토리가 없다면 생성합니다:

```bash
mkdir -p $TARGET/skills $TARGET/rules
```

***

## 2단계: 스킬 선택 및 설치

### 2a: 설치 범위 선택 (핵심 vs 전체)

기본값은 **핵심(Core - 신규 사용자 권장)**입니다. 연구(Research) 우선 워크플로우를 위해 `.agents/skills/*`와 `skills/search-first/`를 복사합니다. 이 번들에는 엔지니어링, 평가, 검증, 보안, 전략적 압축, 프론트엔드 설계 및 Anthropic의 교차 기능 스킬(기사 작성, 콘텐츠 엔진, 시장 조사, 프론트엔드 슬라이드)이 포함됩니다.

`AskUserQuestion`을 사용합니다 (단일 선택):

```
질문: "핵심 스킬만 설치할까요, 아니면 특정 분야/프레임워크 팩을 포함할까요?"
옵션:
  - "핵심 스킬만 (권장)" — "tdd, e2e, evals, verification, research-first, security, frontend patterns, compacting, cross-functional Anthropic skills"
  - "핵심 + 선택한 특정 분야" — "핵심 스킬 설치 후 프레임워크/도메인별 스킬 추가"
  - "특정 분야만" — "핵심 스킬 제외, 특정 프레임워크/도메인 스킬만 설치"
기본값: 핵심 스킬만
```

사용자가 특정 분야를 포함하도록 선택하면 아래 카테고리 선택 단계로 진행하여 선택한 스킬만 포함합니다.

### 2b: 스킬 카테고리 선택

총 27개의 스킬이 4개 카테고리로 나뉩니다. `AskUserQuestion`의 `multiSelect: true` 옵션을 사용합니다:

```
질문: "어떤 스킬 카테고리를 설치하시겠습니까?"
옵션:
  - "프레임워크 및 언어" — "Django, Spring Boot, Go, Python, Java, Frontend, Backend patterns"
  - "데이터베이스" — "PostgreSQL, ClickHouse, JPA/Hibernate patterns"
  - "워크플로우 및 품질" — "TDD, verification, learning, security review, compaction"
  - "모든 스킬" — "사용 가능한 모든 스킬 설치"
```

### 2c: 개별 스킬 확인

선택된 각 카테고리에 대해 아래의 전체 스킬 목록을 보여주고, 사용자가 특정 스킬을 확인하거나 선택 해제하도록 요청하십시오. 목록이 4개 이상인 경우 텍스트로 출력하고 `AskUserQuestion`에서 "나열된 항목 모두 설치" 옵션과 사용자가 특정 이름을 붙여넣을 수 있는 "기타" 옵션을 제공하십시오.

**카테고리: 프레임워크 및 언어 (17개 스킬)**

| 스킬 | 설명 |
|-------|-------------|
| `backend-patterns` | Node.js/Express/Next.js의 백엔드 아키텍처, API 설계, 서버 측 베스트 프랙티스 |
| `coding-standards` | TypeScript, JavaScript, React, Node.js의 공통 코딩 표준 |
| `django-patterns` | Django 아키텍처, DRF 기반 REST API, ORM, 캐싱, 시그널, 미들웨어 |
| `django-security` | Django 보안: 인증, CSRF, SQL 인젝션, XSS 방어 |
| `django-tdd` | pytest-django, factory_boy, 모킹, 커버리지를 활용한 Django 테스트 |
| `django-verification` | Django 검증 루프: 마이그레이션, 린팅, 테스트, 보안 스캔 |
| `frontend-patterns` | React, Next.js, 상태 관리, 성능, UI 패턴 |
| `frontend-slides` | 의존성 없는 HTML 프리젠테이션, 스타일 미리보기 및 PPTX-to-Web 변환 |
| `golang-patterns` | Go다운(Idiomatic) 패턴, 견고한 Go 앱 구축을 위한 규칙 |
| `golang-testing` | Go 테스트: 테이블 구동 테스트, 서브 테스트, 벤치마크, 퍼즈 테스트 |
| `java-coding-standards` | Spring Boot용 Java 코딩 표준: 명명, 불변성, Optional, 스트림 |
| `python-patterns` | Pythonic 관례, PEP 8, 타입 힌트, 베스트 프랙티스 |
| `python-testing` | pytest, TDD, 픽스처, 모킹, 파라미터화를 활용한 Python 테스트 |
| `springboot-patterns` | Spring Boot 아키텍처, REST API, 레이어드 서비스, 캐싱, 비동기 |
| `springboot-security` | Spring Security: 인증/인가, 검증, CSRF, 비밀 정보, 속도 제한 |
| `springboot-tdd` | JUnit 5, Mockito, MockMvc, Testcontainers를 활용한 Spring Boot TDD |
| `springboot-verification` | Spring Boot 검증: 빌드, 정적 분석, 테스트, 보안 스캔 |

**카테고리: 데이터베이스 (3개 스킬)**

| 스킬 | 설명 |
|-------|-------------|
| `clickhouse-io` | ClickHouse 스키마, 쿼리 최격화, 분석, 데이터 엔지니어링 |
| `jpa-patterns` | JPA/Hibernate 엔티티 설계, 관계, 쿼리 최적화, 트랜잭션 |
| `postgres-patterns` | PostgreSQL 쿼리 최적화, 스키마 설계, 인덱싱, 보안 |

**카테고리: 워크플로우 및 품질 (8개 스킬)**

| 스킬 | 설명 |
|-------|-------------|
| `continuous-learning` | 세션에서 재사용 가능한 패턴을 학습된 스킬로 자동 추출 |
| `continuous-learning-v2` | 확신 점수 기반의 본능적 학습, 스킬/명령/에이전트로 진화 |
| `eval-harness` | 평가 기반 개발(EDD)을 위한 공식 평가 프레임워크 |
| `iterative-retrieval` | 서브 에이전트의 컨텍스트 문제를 위한 점진적 컨텍스트 최적화 |
| `security-review` | 보안 체크리스트: 인증, 입력값, 비밀 정보, API, 결제 기능 등 |
| `strategic-compact` | 논리적 간격에서 수동 컨텍스트 압축 제안 |
| `tdd-workflow` | 커버리지 80% 이상의 TDD 강제: 단위, 통합, E2E 테스트 |
| `verification-loop` | 검증 및 품질 루프 패턴 |

**카테고리: 비즈니스 및 콘텐츠 (5개 스킬)**

| 스킬 | 설명 |
|-------|-------------|
| `article-writing` | 메모나 예시, 소스 문서를 활용한 특정 톤의 장문 작성 |
| `content-engine` | 멀티 플랫폼 소셜 콘텐츠, 스크립트 및 콘텐츠 재가공 워크플로우 |
| `market-research` | 출처가 명시된 시장, 경쟁사, 펀드 및 기술 조사 |
| `investor-materials` | 피치 덱, 원페이저, 투자자 메모 및 재무 모델 |
| `investor-outreach` | 맞춤형 투자자 콜드 이메일, 지인 소개 및 후속 팔로업 |

**독립 스킬**

| 스킬 | 설명 |
|-------|-------------|
| `project-guidelines-example` | 프로젝트 전용 스킬 생성을 위한 템플릿 |

### 2d: 설치 실행

선택된 각 스킬에 대해 전체 스킬 디렉토리를 복사합니다:

```bash
cp -r $ECC_ROOT/skills/<skill-name> $TARGET/skills/
```

참고: `continuous-learning` 및 `continuous-learning-v2`는 추가 파일(config.json, 후크, 스크립트)이 있으므로 SKILL.md만 복사하는 것이 아니라 디렉토리 전체를 복사해야 합니다.

***

## 3단계: 규칙(Rules) 선택 및 설치

`AskUserQuestion`의 `multiSelect: true` 옵션을 사용합니다:

```
질문: "어떤 규칙 세트를 설치하시겠습니까?"
옵션:
  - "공통 규칙 (권장)" — "언어에 구속되지 않는 원칙: 코딩 스타일, git 워크플로우, 테스트, 보안 등 (8개 파일)"
  - "TypeScript/JavaScript" — "TS/JS 패턴, 후크, Playwright 테스트 (5개 파일)"
  - "Python" — "Python 패턴, pytest, black/ruff 포매팅 (5개 파일)"
  - "Go" — "Go 패턴, 테이블 구동 테스트, gofmt/staticcheck (5개 파일)"
```

설치를 수행합니다:

```bash
# 공통 규칙 (rules/ 디렉토리에 직접 복사)
cp -r $ECC_ROOT/rules/common/* $TARGET/rules/

# 언어별 규칙 (rules/ 디렉토리에 직접 복사)
cp -r $ECC_ROOT/rules/typescript/* $TARGET/rules/   # 선택 시
cp -r $ECC_ROOT/rules/python/* $TARGET/rules/        # 선택 시
cp -r $ECC_ROOT/rules/golang/* $TARGET/rules/        # 선택 시
```

**중요**: 사용자가 언어별 규칙은 선택했으나 **공통 규칙을 선택하지 않은 경우**, 다음과 같이 경고하십시오:

> "언어별 규칙은 공통 규칙을 확장하여 작동합니다. 공통 규칙을 설치하지 않으면 가이드가 불완전해질 수 있습니다. 공통 규칙도 함께 설치할까요?"

***

## 4단계: 설치 후 검증

설치 완료 후 다음 자동화된 점검을 수행합니다:

### 4a: 파일 존재 확인

설치된 모든 파일을 나열하고 대상 위치에 정상적으로 존재하는지 확인합니다:

```bash
ls -la $TARGET/skills/
ls -la $TARGET/rules/
```

### 4b: 경로 참조 확인

설치된 모든 `.md` 파일 내의 경로 참조를 스캔합니다:

```bash
grep -rn "~/.claude/" $TARGET/skills/ $TARGET/rules/
grep -rn "../common/" $TARGET/rules/
grep -rn "skills/" $TARGET/skills/
```

**프로젝트 수준 설치의 경우**, `~/.claude/` 경로에 대한 참조를 확인합니다:

* 스킬이 `~/.claude/settings.json`을 참조하는 경우 — 설정은 항상 사용자 수준이므로 대개 문제가 없습니다.
* 스킬이 `~/.claude/skills/` 또는 `~/.claude/rules/`를 참조하는 경우 — 프로젝트 수준에만 설치된 경우 작동하지 않을 수 있습니다.
* 스킬이 다른 스킬을 이름으로 참조하는 경우 — 참조되는 스킬도 함께 설치되었는지 확인하십시오.

### 4c: 스킬 간 상호 참조 확인

일부 스킬은 다른 스킬에 의존합니다. 다음 종속성을 확인하십시오:

* `django-tdd`는 `django-patterns`를 참조할 수 있습니다.
* `springboot-tdd`는 `springboot-patterns`를 참조할 수 있습니다.
* `continuous-learning-v2`는 `~/.claude/homunculus/` 디렉토리를 참조합니다.
* `python-testing`은 `python-patterns`를 참조할 수 있습니다.
* `golang-testing`은 `golang-patterns`를 참조할 수 있습니다.
* 언어별 규칙은 대응하는 `common/` 항목을 참조합니다.

### 4d: 문제 보고

발견된 각 문제에 대해 다음을 보고하십시오:

1. **파일**: 문제가 되는 참조가 포함된 파일
2. **줄 번호**: 해당 줄 번호
3. **문제 내용**: 잘못된 점 (예: "~/.claude/skills/python-patterns를 참조하지만 해당 스킬이 설치되지 않음")
4. **수정 제안**: 해결 방법 (예: "python-patterns 스킬 설치" 또는 "경로를 .claude/skills/로 업데이트")

***

## 5단계: 설치된 파일 최적화 (선택 사항)

`AskUserQuestion`을 사용합니다:

```
질문: "프로젝트를 위해 설치된 파일들을 최적화하시겠습니까?"
옵션:
  - "스킬 최적화" — "관련 없는 섹션 제거, 경로 조정, 기술 스택에 맞춤화"
  - "규칙 최적화" — "커버리지 목표 조정, 프로젝트별 패턴 추가, 도구 설정 커스텀"
  - "둘 다 최적화" — "설치된 모든 파일의 전체 최적화"
  - "건너뛰기" — "그대로 유지"
```

### 스킬 최적화 시:

1. 설치된 각 SKILL.md 파일을 읽습니다.
2. (아직 모르는 경우) 사용자의 프로젝트 기술 스택을 물어봅니다.
3. 각 스킬에 대해 관련 없는 섹션의 삭제를 제안합니다.
4. 소스 저장소가 아닌 **설치 대상 경로**의 SKILL.md 파일을 직접 수정합니다.
5. 4단계에서 발견된 경로 문제를 수정합니다.

### 규칙 최적화 시:

1. 설치된 각 규칙 .md 파일을 읽습니다.
2. 사용자의 선호도를 물어봅니다:
   * 테스트 커버리지 목표 (기본 80%)
   * 선호하는 포매팅 도구
   * Git 워크플로우 규칙
   * 보안 요구 사항
3. 설치 대상 경로의 규칙 파일을 직접 수정합니다.

**중요**: 반드시 설치 대상 디렉토리(`$TARGET/`)의 파일만 수정해야 하며, 원본 ECC 저장소(`$ECC_ROOT/`)의 파일은 **절대** 수정하지 마십시오.

***

## 6단계: 설치 요약

`/tmp`에 클론된 저장소를 삭제하여 정리합니다:

```bash
rm -rf /tmp/everything-claude-code
```

그 후 다음과 같이 요약 보고서를 출력합니다:

```
## ECC 설치 완료

### 설치 대상
- 수준: [사용자 수준 / 프로젝트 수준 / 둘 다]
- 경로: [대상 경로]

### 설치된 스킬 ([개수])
- 스킬-1, 스킬-2, 스킬-3, ...

### 설치된 규칙 ([개수])
- common (8개 파일)
- typescript (5개 파일)
- ...

### 검증 결과
- [개수]개 문제 발견, [개수]개 수정 완료
- [남아있는 문제 목록]

### 적용된 최적화
- [변경된 사항 목록 또는 "없음"]
```

***

## 문제 해결 (Troubleshooting)

### "Claude Code가 스킬을 인식하지 못함"

* 스킬 디렉토리에 개별 파일이 아닌 `SKILL.md` 파일이 포함되어 있는지 확인하십시오.
* 사용자 수준: `~/.claude/skills/<skill-name>/SKILL.md`가 있는지 확인하십시오.
* 프로젝트 수준: `.claude/skills/<skill-name>/SKILL.md`가 있는지 확인하십시오.

### "규칙이 작동하지 않음"

* 규칙은 하위 디렉토리가 아닌 단일 파일로 존재해야 합니다: `$TARGET/rules/coding-style.md` (O) vs `$TARGET/rules/common/coding-style.md` (X).
* 규칙 설치 후 Claude Code를 재시작하십시오.

### "프로젝트 수준 설치 후 경로 참조 오류 발생"

* 일부 스킬은 `~/.claude/` 경로를 가정합니다. 4단계 검증을 실행하여 이러한 문제를 찾아 수정하십시오.
* `continuous-learning-v2`의 경우 `~/.claude/homunculus/` 디렉토리는 항상 사용자 수준입니다. 이는 의도된 설계이며 오류가 아닙니다.
