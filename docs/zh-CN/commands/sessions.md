---
description: Claude Code 세션 이력(~/.claude/sessions/)을 관리합니다. 목록 조회, 세션 로드, 별칭(Alias) 설정 및 상세 정보 확인 기능을 제공합니다.
---

# 세션 (Sessions) 명령어

Claude Code의 대화 세션 이력을 관리합니다. 특정 세션을 찾거나, 자주 사용하는 세션에 별명을 붙여 효율적으로 접근할 수 있습니다.

## 사용법

`/sessions [list|load|alias|info|help] [옵션]`

## 주요 작업

### 1. 세션 목록 조회 (list)
저장된 모든 세션의 목록과 메타데이터를 표시합니다.
* `/sessions list`: 모든 세션 출력 (기본값)
* `/sessions list --limit 10`: 최근 10개만 표시
* `/sessions list --date 2026-02-01`: 특정 날짜 세션 필터링
* `/sessions list --search 키워드`: 세션 ID 또는 별칭 검색

### 2. 세션 로드 (load)
세션 ID나 별칭을 사용해 과거 대화 내용을 불러옵니다.
* `/sessions load <ID|별칭>`
* `/sessions load a1b2c3d4`: 단축 ID로 로드
* `/sessions load my-project`: 설정된 별칭으로 로드

### 3. 별칭(Alias) 설정 및 관리
길고 복잡한 세션 ID 대신 기억하기 쉬운 이름을 부여합니다.
* `/sessions alias <ID> <이름>`: 별칭 생성
* `/sessions alias 2026-02-01 daily-sync`: 'daily-sync'라는 별칭 부여
* `/sessions aliases`: 등록된 모든 별칭 목록 확인
* `/sessions unalias <이름>`: 별칭 삭제

### 4. 세션 상세 정보 확인 (info)
세션의 파일 크기, 라인 수, 수정 시각 등 통계 정보를 확인합니다.
* `/sessions info <ID|별칭>`

---

## 실행 예시

```bash
# 세션 목록 확인
/sessions list

# 특정 세션에 별칭 부여
/sessions alias a1b2c3d4 refactoring-job

# 별칭으로 세션 로드
/sessions load refactoring-job

# 세션 상세 정보 확인
/sessions info refactoring-job

# 별칭 목록 확인
/sessions aliases
```

## 참고 사항

* **저장 위치**: 모든 세션은 `~/.claude/sessions/` 경로에 Markdown 파일로 저장됩니다.
* **별칭 관리**: 별칭 설정 정보는 `~/.claude/session-aliases.json` 파일에 별도로 보관됩니다.
* **단축 ID**: 세션 ID 전체를 입력하지 않아도, 중복되지 않는 범위 내에서 앞의 4~8자리만 입력하여 사용할 수 있습니다.
* **불러오기 팁**: 자주 참조해야 하는 설계 문서나 복잡한 리팩토링 세션에는 반드시 별칭을 설정해 두는 것이 좋습니다.

**핵심**: Sessions 명령어를 통해 과거의 작업 문맥을 신속하게 복원하고, 파편화된 대화 기록을 체계적인 지식 베이스로 활용할 수 있습니다.
