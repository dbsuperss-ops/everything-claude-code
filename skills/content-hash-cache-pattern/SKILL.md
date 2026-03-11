---
name: content-hash-cache-pattern
description: SHA-256 콘텐츠 해시를 사용하여 비용이 많이 드는 파일 처리 결과를 캐싱합니다. 경로에 독립적이며 자동 무효화 기능을 갖추고 서비스 레이어가 분리된 패턴입니다.
origin: ECC
---

# 콘텐츠 해시 파일 캐시 패턴 (Content-Hash File Cache Pattern)

SHA-256 콘텐츠 해시를 캐시 키로 사용하여 비용이 많이 드는 파일 처리 작업(PDF 파싱, 텍스트 추출, 이미지 분석 등)의 결과를 캐싱합니다. 경로 기반 캐싱과 달리, 이 방식은 파일 이동이나 이름 변경에도 유지되며 콘텐츠가 변경될 때 자동으로 무효화(invalidate)됩니다.

## 활성화 시점

- 파일 처리 파이프라인(PDF, 이미지, 텍스트 추출) 구축 시
- 처리 비용이 높고 동일한 파일이 반복적으로 처리되는 경우
- `--cache/--no-cache` CLI 옵션이 필요한 경우
- 순수 함수(pure functions)인 기존 로직을 수정하지 않고 캐싱 기능을 추가하고 싶을 때

## 핵심 패턴

### 1. 콘텐츠 해시 기반 캐시 키

파일 경로가 아닌 파일 콘텐츠를 캐시 키로 사용합니다:

```python
import hashlib
from pathlib import Path

_HASH_CHUNK_SIZE = 65536  # 대용량 파일을 위한 64KB 청크

def compute_file_hash(path: Path) -> str:
    """파일 콘텐츠의 SHA-256 해시 계산 (대용량 파일을 위해 청크 단위 처리)."""
    if not path.is_file():
        raise FileNotFoundError(f"파일을 찾을 수 없음: {path}")
    sha256 = hashlib.sha256()
    with open(path, "rb") as f:
        while True:
            chunk = f.read(_HASH_CHUNK_SIZE)
            if not chunk:
                break
            sha256.update(chunk)
    return sha256.hexdigest()
```

**왜 콘텐츠 해시인가?** 파일 이름 변경/이동 시에도 캐시 히트(hit)가 발생하며, 콘텐츠 변경 시 자동으로 무효화됩니다. 인덱스 파일이 따로 필요 없습니다.

### 2. 캐시 항목을 위한 Frozen Dataclass

```python
from dataclasses import dataclass

@dataclass(frozen=True, slots=True)
class CacheEntry:
    file_hash: str
    source_path: str
    document: ExtractedDocument  # 캐싱될 결과물
```

### 3. 파일 기반 캐시 저장소

각 캐시 항목은 `{hash}.json`으로 저장됩니다 — 해시를 통한 O(1) 조회가 가능하며 인덱스 파일이 필요 없습니다.

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
        return None  # 손상된 데이터는 캐시 미스(miss)로 취급
```

### 4. 서비스 레이어 래퍼 (SRP 준수)

처리 함수는 순수하게 유지하고, 캐싱 로직은 별도의 서비스 레이어로 분리합니다.

```python
def extract_with_cache(
    file_path: Path,
    *,
    cache_enabled: bool = True,
    cache_dir: Path = Path(".cache"),
) -> ExtractedDocument:
    """서비스 레이어: 캐시 확인 -> 추출 수행 -> 캐시 쓰기."""
    if not cache_enabled:
        return extract_text(file_path)  # 캐싱 로직이 없는 순수 함수 호출

    file_hash = compute_file_hash(file_path)

    # 캐시 확인
    cached = read_cache(cache_dir, file_hash)
    if cached is not None:
        logger.info("캐시 히트: %s (hash=%s)", file_path.name, file_hash[:12])
        return cached.document

    # 캐시 미스 -> 추출 수행 -> 저장
    logger.info("캐시 미스: %s (hash=%s)", file_path.name, file_hash[:12])
    doc = extract_text(file_path)
    entry = CacheEntry(file_hash=file_hash, source_path=str(file_path), document=doc)
    write_cache(cache_dir, entry)
    return doc
```

## 핵심 설계 결정

| 결정 사항 | 근거 |
|----------|-----------|
| SHA-256 콘텐츠 해시 | 경로 무관, 콘텐츠 변경 시 자동 무효화 |
| `{hash}.json` 파일 명명 | O(1) 조회 성능, 인덱스 파일 불필요 |
| 서비스 레이어 래퍼 | SRP: 추출 로직은 순수하게 유지, 캐싱은 분리된 관심사 |
| 수동 JSON 직렬화 | Frozen Dataclass 직렬화에 대한 완전한 제어권 확보 |
| 손상 시 `None` 반환 | 점진적 기능 저하(Graceful degradation), 다음 실행 시 재처리 |

## 최선 관행 (Best Practices)

- **경로가 아닌 콘텐츠를 해싱하십시오** — 경로는 변하지만 콘텐츠의 정체성은 유지됩니다.
- **대용량 파일은 청크 단위로 해싱하십시오** — 전체 파일을 메모리에 로드하는 것을 피하십시오.
- **처리 함수는 순수하게 유지하십시오** — 처리 로직은 캐싱에 대해 몰라야 합니다.
- **디버깅을 위해 캐시 히트/미스 로그를 남기십시오** — 해시값의 일부를 포함하면 유용합니다.
- **데이터 손상을 유연하게 처리하십시오** — 잘못된 항목은 미스로 취급하고 시스템을 멈추지 마십시오.

## 피해야 할 안티 패턴

```python
# 나쁨: 경로 기반 캐싱 (파일 이동/이름 변경 시 깨짐)
cache = {"/path/to/file.pdf": result}

# 나쁨: 처리 함수 내부에 캐시 로직 추가 (SRP 위반)
def extract_text(path, *, cache_enabled=False, cache_dir=None):
    if cache_enabled:  # 함수가 두 가지 책임을 갖게 됨
        ...
```

**기억하십시오**: 콘텐츠 해시 캐싱은 파일 처리 파이프라인의 성능을 극적으로 향상시킬 수 있습니다. 특히 동일한 파일이 여러 번 처리되는 배치 작업이나 CLI 도구에서 매우 유용합니다.
    
