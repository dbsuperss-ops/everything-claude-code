---
description: "Git 워크플로우: Conventional Commits, PR 프로세스"
alwaysApply: true
---
# Git 워크플로우 (Git Workflow)

## 커밋 메시지 형식
```
<type>: <description>

<optional body>
```

유형(Type): feat, fix, refactor, docs, test, chore, perf, ci

참고: `~/.claude/settings.json`을 통해 전역적으로 서명(Attribution) 기능이 비활성화되어 있습니다.

## 풀 리퀘스트(PR) 프로세스

PR 생성 시 다음 단계를 따르십시오:
1. 전체 커밋 내역을 분석합니다 (마지막 커밋만 보는 것이 아님).
2. `git diff [base-branch]...HEAD` 명령으로 모든 변경 사항을 확인합니다.
3. 포괄적인 PR 요약 문서를 작성합니다.
4. TODO 항목이 포함된 테스트 계획을 포함합니다.
5. 새 브랜치인 경우 `-u` 플래그를 사용하여 푸시합니다.

> Git 작업 이전의 전체 개발 프로세스(계획, TDD, 코드 리뷰)에 대해서는,
> 개발 워크플로우 규칙을 참조하십시오.
