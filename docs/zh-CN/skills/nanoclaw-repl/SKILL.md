---
name: nanoclaw-repl
description: NanoClaw v2(ECC의 claude -p 기반 세션 인식형 REPL)를 조작하고 확장합니다.
origin: ECC
---

# NanoClaw REPL

`scripts/claw.js`를 실행하거나 확장할 때 이 스킬을 활용하십시오.

## 능력 (Capabilities)

* 지속적이고 마크다운 기반의 세션 유지
* `/model` 명령어를 통한 모델 전환
* `/load` 명령어를 통한 동적 스킬 로드
* `/branch` 명령어를 통한 세션 분기
* `/search` 명령어를 통한 세션 간 검색
* `/compact` 명령어를 통한 대화 기록 압축
* `/export` 명령어를 통한 md/json/txt 형식 내보내기
* `/metrics` 명령어를 통한 세션 지표 확인

## 운영 가이드라인

1. 세션을 작업 목표에 집중시키십시오.
2. 위험 부담이 큰 변경 전에는 세션을 분기하십시오.
3. 주요 마일스톤 완료 후에는 기록을 압축하십시오.
4. 공유나 아카이빙 전에는 데이터를 내보내십시오.

## 확장 규칙 (Extension Rules)

* 외부 런타임 의존성을 0으로 유지하십시오.
* 마크다운을 데이터베이스처럼 사용하는 호환성을 유지하십시오.
* 명령 처리기(Command handler)의 결정론적(Deterministic)이고 로컬(Local) 기반의 성격을 유지하십시오.
