---
name: nanoclaw-repl
description: claude -p를 기반으로 구축된 ECC의 외부 의존성 없는 세션 인식 REPL인 NanoClaw v2를 운영하고 확장합니다.
origin: ECC
---

# 나노클로 REPL (NanoClaw REPL)

`scripts/claw.js`를 실행하거나 확장할 때 이 스킬을 사용하십시오.

## 주요 기능

- 마크다운 기반의 지속성 있는 세션 관리
- `/model`: 모델 전환
- `/load`: 동적 스킬 로딩
- `/branch`: 세션 브랜칭 (분기)
- `/search`: 세션 간 검색
- `/compact`: 히스토리 압축
- `/export`: md/json/txt 형식으로 내보내기
- `/metrics`: 세션 지표 확인

## 운영 가이드

1. 각 세션은 특정 작업에 집중하도록 유지하십시오.
2. 위험 부담이 큰 변경 전에는 `/branch`로 브랜치를 만드십시오.
3. 주요 마일스톤 달성 후에는 `/compact`로 히스토리를 압축하십시오.
4. 공유나 아카이빙 전에는 `/export`로 내보내십시오.

## 확장 규칙 (Extension Rules)

- 외부 런타임 의존성을 0으로 유지하십시오.
- "데이터베이스로서의 마크다운" 호환성을 유지하십시오.
- 커맨드 핸들러는 결정론적(Deterministic)이고 로컬에서 동작하도록 유지하십시오.
    
