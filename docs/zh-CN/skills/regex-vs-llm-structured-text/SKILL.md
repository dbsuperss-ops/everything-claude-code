---
name: regex-vs-llm-structured-text
description: 구조화된 텍스트를 파싱할 때 정규표현식(Regex)과 거대 언어 모델(LLM) 중 무엇을 사용할지 결정하기 위한 프레임워크입니다. 정규표현식으로 시작하여 신뢰도가 낮은 예외 케이스에만 LLM을 추가하는 방식을 제안합니다.
origin: ECC
---

# 구조화된 텍스트 파싱을 위한 정규표현식 vs LLM

구조화된 텍스트(퀴즈, 설문지, 영수증, 문서 등)를 파싱하기 위한 실용적인 의사결정 프레임워크입니다. 핵심 통찰은 다음과 같습니다: 정규표현식은 95~98%의 케이스를 매우 저렴하고 결정론적인 방식으로 처리할 수 있습니다. 비용이 많이 드는 LLM 호출은 나머지 예외 케이스(Edge cases)를 위해 아껴두십시오.

## 적용 시점

* 반복되는 패턴이 있는 구조화된 텍스트(질문, 폼, 테이블)를 파싱할 때
* 텍스트 추출 시 정규표현식을 쓸지 LLM을 쓸지 결정해야 할 때
* 두 가지 방식을 결합한 하이브리드 파이프라인을 구축할 때
* 텍스트 처리 과정에서 비용 대비 정확도의 균형을 최적화할 때

## 의사결정 프레임워크

```
텍스트 형식이 일관되고 반복적인가?
├── 예 (90% 이상이 특정 패턴을 따름) → 정규표현식(Regex)으로 시작
│   ├── Regex가 95% 이상 처리함 → 완료, LLM 불필요
│   └── Regex가 95% 미만 처리함 → 예외 케이스에만 LLM 추가
└── 아니오 (자유 형식, 변동성이 매우 큼) → 즉시 LLM 사용
```

## 아키텍처 패턴

```
소스 텍스트 (Source Text)
    │
    ▼
[Regex 파서] ─── 구조 추출 (95~98% 정확도)
    │
    ▼
[텍스트 클리너] ─── 노이즈 제거 (마커, 페이지 번호, 아티팩트 등)
    │
    ▼
[신뢰도 측정기] ─── 신뢰도가 낮은 추출 결과에 플래그 표시
    │
    ├── 높은 신뢰도 (≥0.95) → 즉시 출력
    │
    └── 낮은 신뢰도 (<0.95) → [LLM 검증기] → 최종 출력
```

## 구현 예시

### 1. 정규표현식 파서 (대부분의 케이스 처리)

```python
import re
from dataclasses import dataclass

@dataclass(frozen=True)
class ParsedItem:
    id: str
    text: str
    choices: tuple[str, ...]
    answer: str
    confidence: float = 1.0

def parse_structured_text(content: str) -> list[ParsedItem]:
    """정규표현식 패턴을 사용하여 구조화된 텍스트 파싱"""
    pattern = re.compile(
        r"(?P<id>\d+)\.\s*(?P<text>.+?)\n"
        r"(?P<choices>(?:[A-D]\..+?\n)+)"
        r"Answer:\s*(?P<answer>[A-D])",
        re.MULTILINE | re.DOTALL,
    )
    items = []
    for match in pattern.finditer(content):
        choices = tuple(
            c.strip() for c in re.findall(r"[A-D]\.\s*(.+)", match.group("choices"))
        )
        items.append(ParsedItem(
            id=match.group("id"),
            text=match.group("text").strip(),
            choices=choices,
            answer=match.group("answer"),
        ))
    return items
```

### 2. 신뢰도 측정 (Confidence Scoring)

LLM의 검토가 필요한 항목을 식별합니다.

```python
def score_confidence(item: ParsedItem) -> float:
    """추출 신뢰도를 측정하고 문제 지점을 파악합니다."""
    score = 1.0
    if len(item.choices) < 3: score -= 0.3
    if not item.answer: score -= 0.5
    if len(item.text) < 10: score -= 0.2
    return max(0.0, score)
```

### 3. LLM 검증기 (예외 케이스 전용)

```python
def validate_with_llm(item: ParsedItem, original_text: str, client) -> ParsedItem:
    """신뢰도가 낮은 추출 결과를 수정하기 위해 LLM을 호출합니다."""
    response = client.messages.create(
        model="claude-3-haiku-20240307",  # 검증용으로는 가장 저렴한 모델 권장
        max_tokens=500,
        messages=[{
            "role": "user",
            "content": f"이 텍스트에서 질문, 선택지, 정답을 추출하세요.\n텍스트: {original_text}\n현재 추출 결과: {item}\n수정이 필요하면 JSON으로 반환하고, 정확하면 'CORRECT'라고 답하세요."
        }],
    )
    # LLM 응답을 파싱하여 수정된 항목 반환...
```

## 실제 체감 지표 (Production Metrics)

프로덕션 환경의 퀴즈 파싱 파이프라인(410개 항목 기준) 결과 예시:

| 지표 | 수치 |
|--------|-------|
| 정규표현식 성공률 | 98.0% |
| 낮은 신뢰도 항목(LLM 필요) | 8개 (2.0%) |
| 전체 LLM 사용 대비 비용 절감 | ~95% |
| 전체 파이프라인 정확도 | 99% 이상 |

## 베스트 프랙티스

* **정규표현식으로 시작하십시오** — 완벽하지 않더라도 정규표현식은 개선을 위한 기준점(Baseline)을 제공합니다.
* **신뢰도 측정을 자동화하십시오** — 어떤 항목에 LLM의 도움이 필요한지 프로그래밍 방식으로 판별하십시오.
* **검증에는 가장 저렴한 LLM을 사용하십시오** — Haiku 급 모델로도 충분한 경우가 많습니다.
* **기존 인스턴스를 수정하지 마십시오** — 정제/검증 단계에서는 원본을 유지하고 새로운 인스턴스를 반환하십시오.
* **TDD가 효과적입니다** — 알려진 패턴에 대한 테스트를 먼저 작성하고, 이후 예외 케이스에 대한 테스트를 추가하십시오.

## 피해야 할 안티 패턴

* **과도한 LLM 사용**: 정규표현식으로 95% 이상 처리가능한데 모든 텍스트를 LLM으로 보내는 행위 (비싸고 느림).
* **부적절한 도구 선택**: 형식이 전혀 없고 변동성이 극심한 텍스트에 정규표현식을 고집하는 행위 (LLM이 더 적합함).
* **신뢰도 측정 생략**: 정규표현식이 "잘 작동하겠지"라며 그냥 믿고 넘어가는 행위.
* **에외 케이스 테스트 누락**: 잘못된 형식의 입력, 필드 누락, 인코딩 문제 등을 테스트하지 않는 행위.
