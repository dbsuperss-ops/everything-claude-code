# CLAUDE.md

이 파일은 이 레포지토리의 코드로 작업할 때 Claude Code(claude.ai/code)에 지침을 제공합니다.

## 프로젝트 개요

이 프로젝트는 **Claude Code 플러그인**으로, 바로 사용 가능한 에이전트, 스킬, 후크(hooks), 명령어, 규칙 및 MCP 구성의 모음입니다. 이 프로젝트는 Claude Code를 사용하는 소프트웨어 개발을 위해 검증된 워크플로우를 제공합니다.

## 테스트 실행

```bash
# 모든 테스트 실행
node tests/run-all.js

# 개별 테스트 파일 실행
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

## 아키텍처

프로젝트는 다음과 같은 몇 가지 핵심 구성 요소로 구성됩니다:

- **agents/** - 위임을 위한 전문 서브 에이전트 (planner, code-reviewer, tdd-guide 등)
- **skills/** - 워크플로우 정의 및 도메인 지식 (코딩 표준, 패턴, 테스트)
- **commands/** - 사용자가 호출하는 슬래시 명령어 (/tdd, /plan, /e2e 등)
- **hooks/** - 트리거 기반 자동화 (세션 영속화, 도구 실행 전/후 후크)
- **rules/** - 항상 준수해야 할 가이드라인 (보안, 코딩 스타일, 테스트 요구 사항)
- **mcp-configs/** - 외부 통합을 위한 MCP 서버 구성
- **scripts/** - 후크 및 설정을 위한 크로스 플랫폼 Node.js 유틸리티
- **tests/** - 스크립트 및 유틸리티를 위한 테스트 스위트

## 주요 명령어

- `/tdd` - 테스트 주도 개발 워크플로우
- `/plan` - 구현 계획 수립
- `/e2e` - E2E 테스트 생성 및 실행
- `/code-review` - 품질 리뷰
- `/build-fix` - 빌드 에러 수정
- `/learn` - 세션에서 패턴 추출
- `/skill-create` - git 히스토리에서 스킬 생성

## 개발 참고 사항

- 패키지 매니저 감지: npm, pnpm, yarn, bun (`CLAUDE_PACKAGE_MANAGER` 환경 변수 또는 프로젝트 구성을 통해 설정 가능)
- 크로스 플랫폼: Node.js 스크립트를 통한 Windows, macOS, Linux 지원
- 에이전트 형식: YAML 프론트매터(이름, 설명, 도구, 모델)를 포함한 마크다운
- 스킬 형식: 사용 시기, 작동 방식, 예시로 구성된 명확한 섹션을 가진 마크다운
- 후크 형식: 매처(matcher) 조건과 명령어/알림 후크가 포함된 JSON

## 기여하기

CONTRIBUTING.md의 형식을 따르십시오:
- 에이전트: 프론트매터(이름, 설명, 도구, 모델)가 포함된 마크다운
- 스킬: 명확한 섹션 (사용 시기, 작동 방식, 예시)
- 명령어: 설명 프론트매터가 포함된 마크다운
- 후크: 매처와 후크 배열이 포함된 JSON

파일 명명 규칙: 하이픈으로 연결된 소문자 (예: `python-reviewer.md`, `tdd-workflow.md`)
    
