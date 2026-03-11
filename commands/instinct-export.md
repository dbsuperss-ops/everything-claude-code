---
이름: instinct-export
설명: 프로젝트/글로벌 범위의 본능(instincts)을 파일로 내보냅니다.
명령어: /instinct-export
---

# Instinct Export 명령어 (본능 내보내기)

본능을 공유 가능한 형식으로 내보냅니다. 다음과 같은 경우 유용합니다:
- 팀원과 공유할 때
- 새로운 시스템으로 이전할 때
- 프로젝트 컨벤션에 기여할 때

## 사용법

```
/instinct-export                           # 모든 개인 본능 내보내기
/instinct-export --domain testing          # 테스트 관련 본능만 내보내기
/instinct-export --min-confidence 0.7      # 높은 신뢰도의 본능만 내보내기
/instinct-export --output team-instincts.yaml
/instinct-export --scope project --output project-instincts.yaml
```

## 처리 단계

1. 현재 프로젝트 컨텍스트 감지
2. 선택된 범위에 따라 본능 로드:
   - `project`: 현재 프로젝트만
   - `global`: 글로벌(전체)만
   - `all`: 프로젝트 + 글로벌 통합 (기본값)
3. 필터 적용 (`--domain`, `--min-confidence`)
4. YAML 파일로 내보내기 작성 (출력 경로가 없는 경우 표준 출력(stdout)으로 표시)

## 출력 형식

YAML 파일을 생성합니다:

```yaml
# 본능 내보내기 (Instincts Export)
# 생성일: 2025-01-22
# 출처: 개인(personal)
# 개수: 본능 12개

---
id: prefer-functional-style
trigger: "새로운 함수를 작성할 때"
confidence: 0.8
domain: code-style
source: session-observation
scope: project
project_id: a1b2c3d4e5f6
project_name: my-app
---

# 함수형 스타일 선호 (Prefer Functional Style)

## 행동 (Action)
클래스보다 함수형 패턴을 사용합니다.
```

## 플래그 (Flags)

- `--domain <이름>`: 지정된 도메인만 내보냄
- `--min-confidence <n>`: 최소 신뢰도 임계값
- `--output <파일>`: 출력 파일 경로 (생략 시 표준 출력으로 표시)
- `--scope <project|global|all>`: 내보낼 범위 (기본값: `all`)
