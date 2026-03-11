---
name: instinct-export
description: 프로젝트 또는 전역 범위의 본능(instincts)을 파일로 내보냅니다.
command: /instinct-export
---

# 본능 내보내기 (Instinct Export) 명령어

학습된 본능을 공유 가능한 형식으로 내보냅니다. 다음과 같은 경우에 유용합니다:

* 팀원과 공유할 때
* 새 컴퓨터로 데이터를 옮길 때
* 프로젝트 컨벤션에 기여할 때

## 사용법

```text
/instinct-export                           # 모든 개인 본능 내보내기
/instinct-export --domain testing          # 테스트 관련 본능만 내보내기
/instinct-export --min-confidence 0.7      # 신뢰도가 0.7 이상인 본능만 내보내기
/instinct-export --output team-instincts.yaml
/instinct-export --scope project --output project-instincts.yaml
```

## 처리 절차

1. 현재 프로젝트 컨텍스트를 분석합니다.
2. 선택된 범위에 따라 본능을 로드합니다:
   * `project`: 현재 프로젝트 전용 본능만 포함
   * `global`: 전역 본능만 포함
   * `all`: 프로젝트와 전역 본능을 모두 포함 (기본값)
3. 필터(`--domain`, `--min-confidence`)를 적용합니다.
4. YAML 형식으로 출력합니다. (출력 경로가 지정되지 않으면 표준 출력으로 표시)

## 출력 데이터 형식 (YAML)

```yaml
# Instincts Export
# Generated: 2025-01-22
# Source: personal
# Count: 12 instincts

---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.8
domain: code-style
source: session-observation
scope: project
project_id: a1b2c3d4e5f6
project_name: my-app
---

# Prefer Functional Style

## Action
함수형 패턴을 클래스보다 우선하여 사용하십시오.
```

## 매개변수 (Flags)

* `--domain <이름>`: 지정된 도메인/분야만 내보냄
* `--min-confidence <숫자>`: 최소 신뢰도 기준 (예: 0.7)
* `--output <파일경로>`: 결과를 저장할 파일 경로 (생략 시 화면에 출력)
* `--scope <project|global|all>`: 내보낼 범위 선택 (기본값: `all`)

**핵심**: 내보내기 기능을 통해 개인의 노하우(본능)를 팀의 표준으로 확장하거나, 다양한 개발 환경에서 일관된 에이전트 성능을 유지할 수 있습니다.
