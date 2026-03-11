---
description: NanoClaw v2 시작 — 모델 라우팅, 스킬 핫로드, 브랜칭, 압축, 내보내기 및 지표 측정을 지원하는 ECC의 영구적이고 의존성 없는 REPL입니다.
---

# Claw 명령어

영구적인 마크다운 히스토리와 운영 제어 기능을 갖춘 대화형 AI 에이전트 세션을 시작합니다.

## 사용법

```bash
node scripts/claw.js
```

또는 npm을 통해:

```bash
npm run claw
```

## 환경 변수

| 변수명 | 기본값 | 설명 |
|----------|---------|-------------|
| `CLAW_SESSION` | `default` | 세션 이름 (영문/숫자 + 하이픈) |
| `CLAW_SKILLS` | *(비어있음)* | 시작 시 로드할 쉼표로 구분된 스킬 목록 |
| `CLAW_MODEL` | `sonnet` | 세션에 적용될 기본 모델 |

## REPL 명령어 (REPL Commands)

```text
/help                          도움말 표시
/clear                         현재 세션 히스토리 비우기
/history                       전체 대화 히스토리 출력
/sessions                      저장된 세션 목록 표시
/model [이름]                  모델 확인 또는 설정
/load <스킬-이름>                컨텍스트에 스킬을 즉시(Hot-load) 로드
/branch <세션-이름>              현재 세션을 새로운 브랜치로 분사
/search <검색어>                세션 전체에 걸쳐 검색
/compact                       최근 컨텍스트를 유지하며 이전 대화 압축
/export <md|json|txt> [경로]   세션 내보내기
/metrics                       세션 지표(Metrics) 표시
exit                           종료
```

## 참고 사항

- NanoClaw는 의존성이 없는(zero-dependency) 상태를 유지합니다.
- 세션은 `~/.claude/claw/<세션>.md`에 저장됩니다.
- 압축(Compaction)은 가장 최근의 대화들을 유지하고 압축 헤더를 작성합니다.
- 내보내기(Export)는 마크다운, JSON turns, 일반 텍스트 형식을 지원합니다.
