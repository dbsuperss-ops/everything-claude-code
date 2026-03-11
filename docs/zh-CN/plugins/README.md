# 플러그인 및 마켓플레이스

플러그인은 Claude Code의 기능을 확장하여 새로운 도구와 능력을 추가합니다. 이 가이드는 설치 및 관리에 대한 내용을 다룹니다.

---

## 마켓플레이스 (Marketplace)

마켓플레이스는 설치 가능한 플러그인들이 모여 있는 저장소입니다.

### 마켓플레이스 추가하기

```bash
# 공식 Anthropic 마켓플레이스 추가
claude plugin marketplace add https://github.com/anthropics/claude-plugins-official

# 커뮤니티 마켓플레이스 추가 (예: mgrep)
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep
```

### 추천 마켓플레이스

| 마켓플레이스 | 출처 |
|-------------|--------|
| claude-plugins-official | `anthropics/claude-plugins-official` |
| claude-code-plugins | `anthropics/claude-code` |
| Mixedbread-Grep (@mixedbread-ai) | `mixedbread-ai/mgrep` |

---

## 플러그인 설치하기

```bash
# 플러그인 브라우저 열기
/plugins

# 또는 직접 설치 명령어 사용
claude plugin install typescript-lsp@claude-plugins-official
```

### 추천 플러그인

**개발 및 품질:**
* `typescript-lsp`: TypeScript 지능형 완성 및 분석 지원
* `pyright-lsp`: Python 타입 체크 및 분석 지원
* `code-simplifier`: 코드 리팩토링 및 간소화
* `code-review`: 코드 품질 및 보안 리뷰 자동화
* `security-guidance`: 보안 취약점 점검 및 가이드

**검색 및 워크플로우:**
* `mgrep`: ripgrep보다 향상된 지능형 검색 기능
* `context7`: 실시간 문서 조회 및 컨텍스트 연결
* `commit-commands`: Git 커밋 및 워크플로우 자동화
* `feature-dev`: 기능 개발 프로세스 지원

---

## 빠른 설정 (Quick Start)

```bash
# 필수 마켓플레이스 추가
claude plugin marketplace add https://github.com/anthropics/claude-plugins-official
claude plugin marketplace add https://github.com/mixedbread-ai/mgrep

# '/plugins' 명령어를 입력하여 필요한 플러그인을 선택 설치하십시오.
```

---

## 플러그인 관련 파일 위치

```text
~/.claude/plugins/
├── cache/                    # 다운로드된 플러그인 파일 캐시
├── installed_plugins.json    # 현재 설치된 플러그인 목록
├── known_marketplaces.json   # 추가된 마켓플레이스 정보
└── marketplaces/             # 마켓플레이스별 상세 데이터
```

**핵심**: 플러그인을 활용하여 자신의 개발 스택에 최적화된 Claude Code 환경을 구축하십시오. 특히 LSP 관련 플러그인은 개발 생산성을 비약적으로 높여줍니다.
