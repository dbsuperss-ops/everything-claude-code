---
description: git 히스토리 분석을 통해 스킬 생성
agent: build
---

# 스킬 생성 명령 (Skill Create Command)

git 히스토리를 분석하여 Claude Code 스킬을 생성합니다: $ARGUMENTS

## 임무

1. **커밋 분석** - 히스토리로부터 패턴 인식
2. **패턴 추출** - 공통적인 관행 및 관례 추출
3. **SKILL.md 생성** - 구조화된 스킬 문서 작성
4. **본능(Instincts) 생성** - continuous-learning-v2용 본능 생성

## 분석 프로세스

### 1단계: 커밋 데이터 수집
```bash
# 최근 커밋 내역
git log --oneline -100

# 파일 유형별 커밋 횟수
git log --name-only --pretty=format: | sort | uniq -c | sort -rn

# 가장 많이 수정된 파일
git log --pretty=format: --name-only | sort | uniq -c | sort -rn | head -20
```

### 2단계: 패턴 식별

**커밋 메시지 패턴**:
- 공통 접두사 (feat, fix, refactor)
- 명명 규칙
- 공동 작업자(Co-author) 패턴

**코드 패턴**:
- 파일 구조 관례
- 임포트 구성 방식
- 에러 처리 접근법

**리뷰 패턴**:
- 공통적인 리뷰 피드백
- 반복되는 수정 유형
- 품질 게이트

### 3단계: SKILL.md 생성

```markdown
# [스킬 이름]

## 개요
[이 스킬이 가르치는 내용]

## 패턴

### 패턴 1: [이름]
- 사용 시기
- 구현 방법
- 예시

### 패턴 2: [이름]
- 사용 시기
- 구현 방법
- 예시

## 베스트 프랙티스

1. [프랙티스 1]
2. [프랙티스 2]
3. [프랙티스 3]

## 일반적인 실수

1. [실수 1] - 방지 방법
2. [실수 2] - 방지 방법

## 예시

### 좋은 예시
```[언어]
// 코드 예시
```

### 안티 패턴 (Anti-pattern)
```[언어]
// 하지 말아야 할 것
```
```

### 4단계: 본능(Instincts) 생성

continuous-learning-v2용:

```json
{
  "instincts": [
    {
      "trigger": "[상황]",
      "action": "[대응 조치]",
      "confidence": 0.8,
      "source": "git-history-analysis"
    }
  ]
}
```

## 결과물

생성 내용:
- `skills/[이름]/SKILL.md` - 스킬 문서
- `skills/[이름]/instincts.json` - 본능 모음

---

**팁**: 지속적 학습을 위한 본능도 함께 생성하려면 `/skill-create --instincts` 명령을 실행하십시오.
