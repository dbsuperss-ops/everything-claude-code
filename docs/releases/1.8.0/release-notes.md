# ECC v1.8.0 릴리스 노트

## 포지셔닝

ECC v1.8.0은 단순히 설정 번들이 아닌, 에이전트 하네스 성능 시스템(Agent harness performance system)으로 자리잡았습니다.

## 주요 개선 사항

- 후크 및 라이프사이클 동작의 안정성 강화
- 평가(Eval) 및 루프 작업(Loop operations) 범위 확장
- 실질적인 운영을 위해 NanoClaw 업그레이드
- 크로스 하네스 호환성 향상 (Claude Code, Cursor, OpenCode, Codex 지원)

## 업그레이드 시 주의 사항

1. 로컬 환경에서 기본 후크 프로필 설정을 확인하십시오.
2. `/harness-audit`를 실행하여 프로젝트의 베이스라인 점수를 측정하십시오.
3. `/quality-gate`와 업데이트된 평가 워크플로우를 사용하여 일관성을 유지하십시오.
4. 참조된 기술에 대한 출처 및 라이선스 고지를 확인하십시오: [reference-attribution.md](./reference-attribution.md).
5. 파트너/후원자 미팅 시에는 실시간 배포 지표와 핵심 발표 내용을 활용하십시오: [../business/metrics-and-sponsorship.md](../../business/metrics-and-sponsorship.md).
    
