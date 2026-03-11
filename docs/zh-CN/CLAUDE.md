# CLAUDE.md

이 파일은 Claude Code (claude.ai/code)가 이 저장소의 코드를 처리할 때 필요한 지침을 제공합니다.

## 프로젝트 개요

이 프로젝트는 **Claude Code 플러그인**으로, 운영 환경에서 즉시 사용할 수 있는 에이전트, 스킬, 후크, 명령어, 규칙 및 MCP 구성의 모음입니다. 소프트웨어 개발을 위한 검증된 워크플로우를 제공합니다.

## 테스트 실행

```bash
# 전체 테스트 실행
node tests/run-all.js

# 개별 테스트 파일 실행
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

## 아키텍처

프로젝트는 다음과 같은 핵심 구성 요소로 조직되어 있습니다:

* **agents/** - 위임 작업을 위한 전문 서브 에이전트 (planner, code-reviewer, tdd-guide 등)
* **skills/** - 워크플로우 정의 및 도메인 지식 (코딩 표준, 패턴, 테스트)
* **commands/** - 사용자가 호출하는 슬래시 명령어 (/tdd, /plan, /e2e 등)
* **hooks/** - 트리거 기반 자동화 (세션 유지, 도구 전후 후크)
* **rules/** - 항상 준수해야 할 가이드라인 (보안, 코딩 스타일, 테스트 요구 사항)
* **mcp-configs/** - 외부 통합을 위한 MCP 서버 구성
* **scripts/** - 후크 및 설정을 위한 크로스 플랫폼 Node.js 도구
* **tests/** - 스크립트 및 도구용 테스트 슈트

## 주요 명령어

* `/tdd` - 테스트 주도 개발 워크플로우
* `/plan` - 구현 계획 수립
* `/e2e` - 엔드투엔드 테스트 생성 및 실행
* `/code-review` - 품질 리뷰
* `/build-fix` - 빌드 오류 수정
* `/learn` - 세션에서 패턴 추출
* `/skill-create` - git 이력 기반 스킬 생성

## 개발 참고 사항

* 패키지 관리자 감지: npm, pnpm, yarn, bun (`CLAUDE_PACKAGE_MANAGER` 환경 변수 또는 프로젝트 설정으로 지정 가능)
* 크로스 플랫폼: Node.js 스크립트를 통해 Windows, macOS, Linux 지원
* 에이전트 형식: YAML 프런트매터(이름, 설명, 도구, 모델)가 포함된 마크다운
* 스킬 형식: 명확한 섹션(사용 시기, 작동 방식, 예시)이 포함된 마크다운
* 후크 형식: 매처(matcher) 조건과 명령어/알림 후크가 포함된 JSON

## 기여 가이드

CONTRIBUTING.md의 형식을 따르십시오:

* 에이전트: 프런트매터가 포함된 마크다운 (이름, 설명, 도구, 모델)
* 스킬: 명확한 섹션 (사용 시기, 작동 방식, 예시)
* 명령어: 설명이 포함된 프런트매터 마크다운
* 후크: 매처와 후크 배열이 포함된 JSON

파일 명명: 소문자 및 하이픈 사용 (예: `python-reviewer.md`, `tdd-workflow.md`)
