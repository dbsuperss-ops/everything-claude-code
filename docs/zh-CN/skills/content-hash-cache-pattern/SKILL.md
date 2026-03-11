---
name: content-hash-cache-pattern
description: SHA-256 콘텐츠 해시를 사용하여 비용이 많이 드는 파일 처리 결과를 캐싱합니다. 경로와 무관하고 자동 실효 기능이 포함된 서비스 레이어 분리 패턴입니다.
origin: ECC
---

# 콘텐츠 해시 파일 캐시 패턴

SHA-256 콘텐츠 해시를 캐시 키로 사용하여 비용이 많이 드는 파일 처리 작업(PDF 파싱, 텍스트 추출, 이미지 분석 등)의 결과를 캐싱합니다. 경로 기반 캐싱과 달리, 이 방식은 파일을 이동하거나 이름을 바꿔도 유효하며 내용이 변경되면 자동으로 캐시가 실효(Invalidation)됩니다.

## 적용 시점

* 파일 처리 파이프라인(PDF, 이미지, 텍스트 추출 등)을 구축할 때
* 처리 비용은 높지만 동일한 파일이 반복적으로 처리되는 경우
* `--cache/--no-cache`와 같은 CLI 옵션이 필요한 경우
* 기존 순수 함수(Pure function)를 수정하지 않고 캐싱 기능을 추가하고 싶을 때

## 핵심 패턴

### 1. 콘텐츠 해시 기반 캐시 키
파일 경로가 아닌 파일 내용 자체를 해싱하여 캐시 키로 사용합니다.

```python
import hashlib
from pathlib import Path

_HASH_CHUNK_SIZE = 65536  # 대용량 파일을 위한 64KB 청크

def compute_file_hash(path: Path) -> str:
    """파일 내용의 SHA-256 해시를 계산합니다 (대용량 파일 대응)."""
    if not path.is_file():
        raise FileNotFoundError(f"파일을 찾을 수 없습니다: {path}")
    sha256 = hashlib.sha256()
    with open(path, "rb") as f:
        while True:
            chunk = f.read(_HASH_CHUNK_SIZE)
            if not chunk:
                break
            sha256.update(chunk)
    return sha256.hexdigest()
```

**왜 콘텐츠 해시인가?** 파일 이름을 바꾸거나 이동해도 캐시 적중(Hit)이 발생합니다. 내용이 조금이라도 바뀌면 자동으로 캐시가 무효화됩니다. 인덱스 파일이 따로 필요하지 않습니다.

### 2. 캐시 항목을 위한 불변 데이터 클래스 (Frozen Dataclass)

```python
from dataclasses import dataclass

@dataclass(frozen=True, slots=True)
class CacheEntry:
    file_hash: str
    source_path: str
    document: ExtractedDocument  # 캐싱될 결과물
```

### 3. 파일 기반 캐시 저장소
각 캐시 항목은 `{hash}.json` 형태로 저장됩니다. 해시값으로 인덱스 파일 없이 O(1) 조회가 가능합니다.

```python
import json
from typing import Any

def write_cache(cache_dir: Path, entry: CacheEntry) -> None:
    cache_dir.mkdir(parents=True, exist_ok=True)
    cache_file = cache_dir / f"{entry.file_hash}.json"
    data = serialize_entry(entry)
    cache_file.write_text(json.dumps(data, ensure_ascii=False), encoding="utf-8")

def read_cache(cache_dir: Path, file_hash: str) -> CacheEntry | None:
    cache_file = cache_dir / f"{file_hash}.json"
    if not cache_file.is_file():
        return None
    try:
        raw = cache_file.read_text(encoding="utf-8")
        data = json.loads(raw)
        return deserialize_entry(data)
    except (json.JSONDecodeError, ValueError, KeyError):
        return None  # 데이터 손상 시 캐시 미스(Miss)로 처리
```

### 4. 서비스 레이어 래퍼 (단일 책임 원칙)
실제 데이터 처리 함수의 순수성을 유지하십시오. 캐싱은 별도의 서비스 레이어로 추가합니다.

```python
def extract_with_cache(
    file_path: Path,
    *,
    cache_enabled: bool = True,
    cache_dir: Path = Path(".cache"),
) -> ExtractedDocument:
    """서비스 레이어: 캐시 확인 -> 추출 실행 -> 결과 저장."""
    if not cache_enabled:
        return extract_text(file_path)  # 순수 함수 호출, 캐시 로직 없음

    file_hash = compute_file_hash(file_path)

    # 캐시 확인
    cached = read_cache(cache_dir, file_hash)
    if cached is not None:
        logger.info("캐시 적중(Hit): %s (hash=%s)", file_path.name, file_hash[:12])
        return cached.document

    # 캐시 미스 -> 추출 -> 저장
    logger.info("캐시 미스(Miss): %s (hash=%s)", file_path.name, file_hash[:12])
    doc = extract_text(file_path)
    entry = CacheEntry(file_hash=file_hash, source_path=str(file_path), document=doc)
    write_cache(cache_dir, entry)
    return doc
```

## 주요 설계 원칙

| 결정 사항 | 이유 |
|----------|-----------|
| SHA-256 콘텐츠 해시 | 경로 무관성 확보, 내용 변경 시 자동 무효화 |
| `{hash}.json` 파일 명명 | 별도 인덱스 없이 O(1) 조회 가능 |
| 서비스 레이어 래퍼 | 단일 책임 원칙(SRP): 추출 로직은 순수하게 유지, 캐싱은 별도 관심사로 분리 |
| 수동 JSON 직렬화 | 불변 데이터 클래스의 직렬화 방식 정밀 제어 |
| 손상 시 `None` 반환 | 점진적 기능 저하(Graceful degradation), 다음 실행 시 재처리 |
| `cache_dir.mkdir(parents=True)` | 첫 쓰기 발생 시점에 디렉토리 생성(Lazy creation) |

## 베스트 프랙티스

* **경로가 아닌 내용(Contents)을 해싱하십시오** — 경로는 변하지만 내용 식별자는 변하지 않습니다.
* **대용량 파일 해싱 시 청크(Chunk) 단위로 처리하십시오** — 파일 전체를 메모리에 올리지 마십시오.
* **데이터 처리 함수를 순수하게 유지하십시오** — 해당 함수들은 캐시의 존재를 몰라야 합니다.
* **캐시 적중/미스 여부를 로깅하십시오** — 디버깅을 위해 단축된 해시값을 함께 출력하십시오.
* **데이터 손상을 유연하게 처리하십시오** — 유효하지 않은 캐시 파일은 미스로 간주하고 프로그램을 종료시키지 마십시오.

## 피해야 할 안티 패턴

```python
# ❌ 나쁜 예: 경로 기반 캐싱 (파일 이동/이름 변경 시 캐시 깨짐)
cache = {"/path/to/file.pdf": result}

# ❌ 나쁜 예: 처리 함수 내부에 캐시 로직 추가 (단일 책임 원칙 위반)
def extract_text(path, *, cache_enabled=False, cache_dir=None):
    if cache_enabled:  # 이제 이 함수는 두 가지 책임을 갖게 됨
        ...

# ❌ 나쁜 예: 복잡하게 중첩된 데이터 클래스에 dataclasses.asdict() 사용
# (복잡한 타입에서 문제가 발생할 수 있음)
data = dataclasses.asdict(entry)  # 대신 수동 직렬화(Manual serialization) 권장
```

## 적용 가능한 케이스
* 파일 처리 파이프라인 (PDF 파싱, OCR, 텍스트 추출, 이미지 분석 등)
* `--cache/--no-cache` 옵션이 있는 CLI 도구
* 여러 번 실행되는 배치 프로세스 내의 중복 파일 처리
* 기존 순수 함수를 건드리지 않고 성능을 개선하고 싶을 때

## 적용하기 어려운 케이스
* 항상 최신 상태여야 하는 실시간 데이터 스트림
* 캐시 결과물이 너무 거대한 경우 (스트리밍 방식 고려 필요)
* 결과가 파일 내용 외의 다른 파라미터(예: 추출 설정값 등)에 의존하는 경우
