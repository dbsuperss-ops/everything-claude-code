---
name: cost-aware-llm-pipeline
description: LLM API 사용 비용 최적화 패턴 — 작업 복잡도에 따른 모델 라우팅, 예산 추적, 재시도 로직 및 프롬프트 캐싱 가이드입니다.
origin: ECC
---

# 비용 인식 LLM 파이프라인 (Cost-Aware LLM Pipeline)

품질을 유지하면서 LLM API 비용을 제어하기 위한 패턴입니다. 모델 라우팅, 예산 추적, 재시도 로직, 프롬프트 캐싱을 하나의 조합 가능한 파이프라인으로 결합합니다.

## 활성화 시점

- LLM API(Claude, GPT 등)를 호출하는 애플리케이션 구축 시
- 복잡도가 다양한 대량의 항목을 처리할 때
- API 지출 예산 범위 내에서 운영해야 할 때
- 복잡한 작업에서 품질 저하 없이 비용을 최적화하고 싶을 때

## 핵심 개념

### 1. 작업 복잡도에 따른 모델 라우팅

단순한 작업에는 저렴한 모델을 사용하고, 복잡한 작업에만 고성능 모델(비싼 모델)을 사용하도록 자동화합니다.

```python
MODEL_SONNET = "claude-sonnet-4-6"
MODEL_HAIKU = "claude-haiku-4-5-20251001"

_SONNET_TEXT_THRESHOLD = 10_000  # 글자 수 임계값
_SONNET_ITEM_THRESHOLD = 30      # 항목 수 임계값

def select_model(text_length: int, item_count: int) -> str:
    """작업 복잡도에 따라 모델 선택."""
    if text_length >= _SONNET_TEXT_THRESHOLD or item_count >= _SONNET_ITEM_THRESHOLD:
        return MODEL_SONNET  # 복잡한 작업
    return MODEL_HAIKU  # 단순한 작업 (3-4배 저렴)
```

### 2. 불변성(Immutable) 비용 추적

Frozen Dataclass를 사용하여 누적 지출을 추적합니다. 각 API 호출은 상태를 수정하지 않고 새로운 추적 객체를 반환합니다.

```python
@dataclass(frozen=True, slots=True)
class CostTracker:
    budget_limit: float = 1.00
    records: tuple[CostRecord, ...] = ()

    def add(self, record: CostRecord) -> "CostTracker":
        """데이터를 추가한 새로운 추적 객체 반환 (원본 수정 없음)."""
        return CostTracker(records=(*self.records, record))

    @property
    def over_budget(self) -> bool:
        return self.total_cost > self.budget_limit
```

### 3. 세밀한 재시도 로직 (Narrow Retry)

일시적인 에러에 대해서만 재시도하고, 인증 에러나 잘못된 요청(Bad request)의 경우에는 즉시 실패 처리합니다.

```python
# 일시적 에러(네트워크, 속도 제한, 서버 내부 에러)만 재시도 대상
_RETRYABLE_ERRORS = (APIConnectionError, RateLimitError, InternalServerError)
```

### 4. 프롬프트 캐싱 (Prompt Caching)

긴 시스템 프롬프트를 캐싱하여 매 요청마다 다시 전송하는 비용을 절감합니다.

```python
"cache_control": {"type": "ephemeral"}  # 이 부분을 캐싱함
```

## 최선 관행 (Best Practices)

- **가장 저렴한 모델부터 시작하십시오**: 복잡도 임계값을 넘을 때만 고성능 모델로 라우팅하십시오.
- **명시적인 예산 한도를 설정하십시오**: 대량의 작업을 처리하기 전에 한도를 설정하여 예상치 못한 과금을 방지하십시오.
- **모델 선택 결정을 기록하십시오**: 실제 데이터를 바탕으로 라우팅 임계값을 조정할 수 있습니다.
- **1024 토큰 이상의 시스템 프롬프트는 캐싱을 사용하십시오**: 비용과 지연 시간(Latency)을 모두 줄일 수 있습니다.
- **일시적 실패에만 재시도하십시오**: 인증이나 데이터 검증 에러에 재시도하는 것은 예산 낭비입니다.

## 피해야 할 안티 패턴

- 모든 요청에 복잡도와 상관없이 가장 비싼 모델을 사용하는 것
- 모든 에러에 재시도하는 것 (영구적인 실패에도 예산을 낭비하게 됨)
- 모델 이름을 코드 곳곳에 하드코딩하는 것 (설정 파일이나 상수를 사용하십시오)
- 반복되는 긴 프롬프트에 캐싱을 적용하지 않는 것

**기억하십시오**: 비용 인식 파이프라인은 특히 대량 처리 작업에서 성능과 경제성의 균형을 맞추는 데 필수적입니다.
    
