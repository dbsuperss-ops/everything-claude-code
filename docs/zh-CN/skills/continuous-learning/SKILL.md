---
name: continuous-learning
description: Claude Code 세션에서 재사용 가능한 패턴을 자동으로 추출하여 나중에 사용할 수 있도록 학습된 스킬로 저장합니다.
origin: ECC
---

# 지속적 학습(Continuous Learning) 스킬

Claude Code 세션이 종료될 때 이를 자동으로 평가하여, 나중에 재사용할 수 있는 패턴을 추출하고 '학습된 스킬(Learned Skill)'로 저장합니다.

## 적용 시점

* Claude Code 세션에서 패턴을 자동으로 추출하도록 설정할 때
* 세션 평가를 위한 중단 훅(Stop hook)을 구성할 때
* `~/.claude/skills/learned/` 경로에 저장된 학습된 스킬을 검토하거나 정리할 때
* 추출 임계값(Threshold)이나 패턴 카테고리를 조정할 때
* v1(본 방식)과 v2(본능 기반 방식)를 비교할 때

## 작동 원리

이 스킬은 각 세션이 끝날 때 **중단 훅(Stop hook)**으로 실행됩니다:

1. **세션 평가**: 세션에 포함된 메시지 수가 충분한지 확인합니다 (기본값: 10개 이상).
2. **패턴 탐지**: 세션 기록에서 추출 가능한 유용한 패턴을 식별합니다.
3. **스킬 추출**: 유용한 패턴을 `~/.claude/skills/learned/` 경로에 파일로 저장합니다.

## 설정

`config.json` 파일을 편집하여 동작을 커스터마이징할 수 있습니다:

```json
{
  "min_session_length": 10,
  "extraction_threshold": "medium",
  "auto_approve": false,
  "learned_skills_path": "~/.claude/skills/learned/",
  "patterns_to_detect": [
    "error_resolution",
    "user_corrections",
    "workarounds",
    "debugging_techniques",
    "project_specific"
  ],
  "ignore_patterns": [
    "simple_typos",
    "one_time_fixes",
    "external_api_issues"
  ]
}
```

## 패턴 유형

| 패턴 | 설명 |
|---------|-------------|
| `error_resolution` | 특정 에러가 어떻게 해결되었는지에 대한 패턴 |
| `user_corrections` | 사용자의 교정 및 피드백으로부터 얻은 패턴 |
| `workarounds` | 프레임워크나 라이브러리의 특이 케이스에 대한 우회 해결책 |
| `debugging_techniques` | 효과적이었던 디버깅 방법론 |
| `project_specific` | 특정 프로젝트만의 고유한 규칙이나 관례 |

## 훅(Hook) 설정

`~/.claude/settings.json` 파일에 다음 내용을 추가하십시오:

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
      }]
    }]
  }
}
```

## 왜 중단 훅(Stop hook)을 사용하나요?

* **경량화**: 세션이 완전히 끝날 때 딱 한 번만 실행됩니다.
* **비차단형(Non-blocking)**: 개별 메시지를 주고받을 때 지연 시간을 추가하지 않습니다.
* **전체 컨텍스트**: 세션 전체의 대화 기록에 접근하여 종합적인 분석이 가능합니다.

## 관련 정보

* [가이드 전문](https://x.com/affaanmustafa/status/2014040193557471352) - 지속적 학습 섹션 참조
* `/learn` 명령 - 세션 도중 수동으로 패턴 추출하기

***

## 비교 분석 (연구: 2025년 1월)

### Homunculus v2와의 비교

더 복잡한 접근 방식을 사용하는 Homunculus v2와의 차이점입니다:

| 기능 | 현재 방식 (v1) | Homunculus v2 |
|---------|--------------|---------------|
| 관찰 시점 | 중단 훅 (세션 종료 시) | Pre/PostToolUse 훅 (100% 신뢰도) |
| 분석 주체 | 메인 컨텍스트 | 백그라운드 에이전트 (Haiku) |
| 분석 단위 | 완성된 형태의 스킬 | 원자 단위의 '본능(Instinct)' |
| 가중치 | 없음 | 0.3~0.9 가중치 부여 |
| 진화 과정 | 즉시 스킬로 생성 | 본능 → 군집(Cluster) → 스킬/명령/에이전트 |
| 공유 가능성 | 없음 | 본능 내보내기/가져오기 가능 |

**Homunculus 프로젝트의 핵심 견해:**
> "v1은 관찰을 위해 '스킬' 기능에 의존합니다. 스킬은 확률적(약 50~80% 트리거)입니다. v2는 관찰을 위해 '훅'을 사용하여 100% 신뢰도를 확보하고, '본능'을 학습된 행동의 원자 단위로 사용합니다."

### 잠재적 v2 개선 사항

1. **본능 기반 학습** - 가중치가 포함된 더 작고 원자적인 행동 단위 분석
2. **백그라운드 관찰자** - Haiku 에이전트를 통한 병렬 분석 수행
3. **신뢰도 감쇠(Confidence Decay)** - 반증되는 사례가 나오면 본능의 신뢰도 수치 감소
4. **도메인 태깅** - 코드 스타일, 테스트, git, 디버깅 등으로 분류
5. **진화 경로** - 관련된 본능들을 모아 하나의 스킬이나 명령어로 패키징

상세 내용은 `docs/continuous-learning-v2-spec.md` 명세서를 참조하십시오.
