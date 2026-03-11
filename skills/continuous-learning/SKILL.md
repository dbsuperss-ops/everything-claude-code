---
name: continuous-learning
description: Claude Code 세션에서 재사용 가능한 패턴을 자동으로 추출하여 미래에 사용할 수 있는 학습된 스킬(Learned skills)로 저장합니다.
origin: ECC
---

# 지속적 학습 스킬 (Continuous Learning Skill)

Claude Code 세션이 종료될 때 자동으로 세션을 평가하여, 학습된 스킬로 저장할 수 있는 재사용 가능한 패턴을 추출합니다.

## 활성화 시점

- Claude Code 세션에서 자동 패턴 추출 설정을 할 때
- 세션 평가를 위한 Stop 훅(hook)을 구성할 때
- `~/.claude/skills/learned/`에 저장된 학습된 스킬을 검토하거나 관리할 때
- 추출 임계값(Threshold)이나 패턴 카테고리를 조정할 때
- v1(현재 방식)과 v2(본능 기반 학습) 방식을 비교할 때

## 작동 방식

이 스킬은 각 세션이 끝날 때 **Stop 훅**으로 실행됩니다:

1. **세션 평가**: 세션에 충분한 메시지가 있는지 확인합니다 (기본값: 10개 이상).
2. **패턴 감지**: 세션에서 추출 가능한 패턴을 식별합니다.
3. **스킬 추출**: 유용한 패턴을 `~/.claude/skills/learned/` 폴더에 저장합니다.

## 설정

`config.json` 파일을 수정하여 설정을 맞춤화하십시오:

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
| `error_resolution` | 특정 에러가 어떻게 해결되었는지 |
| `user_corrections` | 사용자의 교정 사항에서 파생된 패턴 |
| `workarounds` | 프레임워크나 라이브러리의 특이점(Quirk)에 대한 해결책 |
| `debugging_techniques` | 효과적인 디버깅 접근 방식 |
| `project_specific` | 프로젝트 전용 컨벤션 |

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

## 관련 항목

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 지속적 학습 섹션
- `/learn` 명령어 - 세션 중간에 수동으로 패턴 추출 수행

---

## 비교 분석 (Research: 2025년 1월)

### vs Homunculus

Homunculus v2는 더 정교한 접근 방식을 취합니다:

| 기능 | 현재 방식 (v1) | Homunculus v2 |
|---------|--------------|---------------|
| 관찰 시점 | Stop 훅 (세션 종료 시) | Pre/PostToolUse 훅 (100% 신뢰도) |
| 분석 주체 | 메인 컨텍스트 | 백그라운드 에이전트 (Haiku) |
| 세밀도 | 전체 스킬 단위 | 원자적 "본능(Instincts)" 단위 |
| 확신도 점수 | 없음 | 0.3-0.9 가중치 부여 |
| 진화 경로 | 스킬로 직접 저장 | 본능 → 클러스터링 → 스킬/명령어/에이전트 |
| 공유 가능성 | 없음 | 본능 내보내기/가져오기 가능 |

**Homunculus의 핵심 통찰:**
> "v1은 관찰을 위해 스킬에 의존했습니다. 스킬은 확률적입니다—약 50-80% 확률로 작동합니다. v2는 관찰을 위해 훅을 사용(100% 신뢰도)하며, 학습된 행동의 원자적 단위로 '본능(Instincts)'을 사용합니다."

상세 사양은 `docs/continuous-learning-v2-spec.md`를 참조하십시오.
    
