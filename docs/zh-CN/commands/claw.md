---
description: NanoClaw v2(ECC의 영구적이고 의존성 없는 REPL)를 실행합니다. 모델 라우팅, 스킬 핫 로드, 세션 분기, 압축, 내보내기 및 지표 기능을 제공합니다.
---

# Claw 명령어

영구적인 Markdown 기록과 강력한 작업 제어 기능을 갖춘 대화형 AI 에이전트 세션을 시작합니다.

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
| `CLAW_SKILLS` | *(없음)* | 시작 시 로드할 스킬 목록 (쉼표로 구분) |
| `CLAW_MODEL` | `sonnet` | 세션의 기본 모델 설정 |

## REPL 전용 명령어 (세션 내부)

```text
/help                          도움말 표시
/clear                         현재 세션 이력 삭제
/history                       전체 대화 이력 출력
/sessions                      저장된 세션 목록 보기
/model [이름]                  모델 확인 또는 설정
/load <스킬명>                 컨텍스트에 스킬 즉시(Hot-load) 로드
/branch <세션명>               현재 세션에서 새로운 분기 생성
/search <검색어>               세션 전체에서 검색
/compact                       오래된 턴을 압축하고 최근 컨텍스트 유지
/export <md|json|txt> [경로]   세션 내보내기
/metrics                       세션 통계/지표 표시
exit                           종료
```

## 주요 특징

* **NanoClaw**는 외부 의존성 없이 독립적으로 작동합니다.
* 모든 세션은 `~/.claude/claw/<세션명>.md` 파일로 저장됩니다.
* **압축(Compact)** 기능은 최근 대화 내용은 보존하면서 이전 내용은 요약하여 헤더에 기록합니다.
* **내보내기(Export)**는 Markdown, JSON(턴 단위), 일반 텍스트 형식을 지원합니다.

**핵심**: Claw는 Claude Code가 없는 환경에서도 ECC의 핵심 기능을 사용할 수 있게 해주는 가볍고 강력한 도구입니다. 세션 관리와 토큰 최적화에 탁월합니다.
