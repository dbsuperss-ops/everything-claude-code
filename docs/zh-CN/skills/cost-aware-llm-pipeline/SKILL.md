---
name: cost-aware-llm-pipeline
description: LLM API 사용 비용 최적화 패턴입니다. 작업 복잡도 기반 모델 라우팅, 예산 추적, 재시도 로직 및 프롬프트 캐싱을 포함합니다.
origin: ECC
---

# 비용 인식형 LLM 파이프라인

품질을 유지하면서 LLM API 비용을 제어하는 패턴입니다. 모델 라우팅, 예산 추적, 재시도 로직, 프롬프트 캐싱을 결합하여 조립 가능한 파이프라인을 구축합니다.

## 적용 시점

* LLM API(Claude, GPT 등)를 호출하는 애플리케이션을 구축할 때
* 복잡도가 다양한 대량의 항목을 처리할 때
* API 비용 지출을 사전에 설정한 예산 범위 내로 유지해야 할 때
* 복잡한 작업에서 품질 저하 없이 비용을 최적화하고 싶을 때

## 핵심 개념

### 1. 작업 복잡도 기반 모델 라우팅
간단한 작업에는 저렴한 모델을 선택하고, 복잡한 작업에만 고비용 모델을 할당하여 비용을 절감합니다.

```python
MODEL_SONNET = "claude-3-5-sonnet-latest"
MODEL_HAIKU = "claude-3-haiku-20240307"

_SONNET_TEXT_THRESHOLD = 10_000  # 글자 수 기준
_SONNET_ITEM_THRESHOLD = 30     # 항목 개수 기준

def select_model(
    text_length: int,
    item_count: int,
    force_model: str | None = None,
) -> str:
    """작업 복잡도에 따라 적절한 모델을 선택합니다."""
    if force_model is not None:
        return force_model
    if text_length >= _SONNET_TEXT_THRESHOLD or item_count >= _SONNET_ITEM_THRESHOLD:
        return MODEL_SONNET  # 복잡한 작업
    return MODEL_HAIKU  # 간단한 작업 (약 3~4배 저렴)
```

### 2. 불변(Immutable) 비용 추적
불변 데이터 클래스를 사용하여 누적 지출을 추적합니다. 각 API 호출은 상태를 변경하는 대신 새로운 추적기 객체를 반환합니다.

```python
from dataclasses import dataclass

@dataclass(frozen=True, slots=True)
class CostRecord:
    model: str
    input_tokens: int
    output_tokens: int
    cost_usd: float

@dataclass(frozen=True, slots=True)
class CostTracker:
    budget_limit: float = 1.00
    records: tuple[CostRecord, ...] = ()

    def add(self, record: CostRecord) -> "CostTracker":
        """새로운 기록이 추가된 새 추적기 객체를 반환합니다(원본 수정 없음)."""
        return CostTracker(
            budget_limit=self.budget_limit,
            records=(*self.records, record),
        )

    @property
    def total_cost(self) -> float:
        return sum(r.cost_usd for r in self.records)

    @property
    def over_budget(self) -> bool:
        return self.total_cost > self.budget_limit
```

### 3. 세밀한 재시도 로직
일시적인(Transient) 에러에 대해서만 재시도합니다. 인증 에러나 잘못된 요청(BadRequest)은 즉시 실패 처리하여 리소스를 아낍니다.

```python
from anthropic import (
    APIConnectionError,
    InternalServerError,
    RateLimitError,
)

_RETRYABLE_ERRORS = (APIConnectionError, RateLimitError, InternalServerError)
_MAX_RETRIES = 3

def call_with_retry(func, *, max_retries: int = _MAX_RETRIES):
    """일시적 에러에 대해서만 재시도하며, 그 외의 경우에는 즉시 실패 처리합니다."""
    for attempt in range(max_retries):
        try:
            return func()
        except _RETRYABLE_ERRORS:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)  # 지수 백오프(Exponential backoff)
    # AuthenticationError, BadRequestError 등은 즉시 에러 발생
```

### 4. 프롬프트 캐싱 (Prompt Caching)
길고 반복되는 시스템 프롬프트를 캐싱하여 매 요청마다 다시 전송하는 비용을 줄입니다.

```python
messages = [
    {
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": system_prompt,
                "cache_control": {"type": "ephemeral"},  # 이 부분 캐싱
            },
            {
                "type": "text",
                "text": user_input,  # 변경되는 입력 부분
            },
        ],
    }
]
```

## 파이프라인 결합 예시

네 가지 기술을 하나의 파이프라인 함수로 결합합니다:

```python
def process(text: str, config: Config, tracker: CostTracker) -> tuple[Result, CostTracker]:
    # 1. 모델 라우팅
    model = select_model(len(text), estimated_items, config.force_model)

    # 2. 예산 확인
    if tracker.over_budget:
        raise BudgetExceededError(tracker.total_cost, tracker.budget_limit)

    # 3. 재시도 로직 + 캐싱을 적용하여 호출
    response = call_with_retry(lambda: client.messages.create(
        model=model,
        messages=build_cached_messages(system_prompt, text),
    ))

    # 4. 비용 추적 (불변 객체 반환)
    record = CostRecord(model=model, input_tokens=..., output_tokens=..., cost_usd=...)
    tracker = tracker.add(record)

    return parse_result(response), tracker
```

## 가격 참조 (2025-2026 기준)

| 모델 | 입력 ($/1M 토큰) | 출력 ($/1M 토큰) | 상대 비용 |
|-------|---------------------|----------------------|---------------|
| Haiku 4.5 | $0.80 | $4.00 | 1x |
| Sonnet 4.6 | $3.00 | $15.00 | ~4x |
| Opus 4.5 | $15.00 | $75.00 | ~19x |

## 베스트 프랙티스

* **가장 저렴한 모델부터 시작하십시오.** 복잡도 임계값을 넘을 때만 고비용 모델로 라우팅합니다.
* **배치 작업 전 명확한 예산 한도를 설정하십시오.** 비용 초과 전에 미리 실패(Fail fast)하는 것이 안전합니다.
* **모델 선택 결정을 로깅하십시오.** 실제 데이터를 바탕으로 임계값을 최적으로 조정할 수 있습니다.
* **1,024 토큰 이상의 시스템 프롬프트에는 프롬프트 캐싱을 사용하십시오.** 비용뿐만 아니라 지연 시간(Latency)도 줄어듭니다.
* **인증이나 검증 에러 발생 시 재시도하지 마십시오.** 네트워크 문제, 속도 제한, 서버 에러 같은 일시적 장애에만 재시도를 적용합니다.

## 피해야 할 안티 패턴

* 작업 복잡도와 상관없이 무조건 최고가 모델을 사용함
* 모든 에러에 대해 재시도함 (해결 불가능한 오류에 예산을 낭비함)
* 비용 추적 객체의 상태를 직접 수정함 (디버깅 및 감사가 어려워짐)
* 코드 곳곳에 모델 이름을 하드코딩함 (상수나 설정 파일 사용 권장)
* 반복되는 시스템 프롬프트에 프롬프트 캐싱을 적용하지 않음

## 적용 가능한 시나리오

* Claude, OpenAI 등 LLM API를 호출하는 모든 애플리케이션
* 비용이 급격히 누적될 수 있는 대규모 배치 처리 파이프라인
* 지능형 라우팅이 필요한 멀티 모델 아키텍처
* 예산 가드레일이 필요한 프로덕션 시스템
