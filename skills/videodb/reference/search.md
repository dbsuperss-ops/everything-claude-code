# 검색 및 인덱싱 가이드 (Search & Indexing Guide)

검색 기능을 통해 자연어 쿼리, 정확한 키워드 또는 시각적 장면 설명을 사용하여 비디오 내부의 특정 순간을 찾을 수 있습니다.

## 전제 조건

비디오를 검색하려면 먼저 **인덱싱(Indexing)** 작업이 완료되어야 합니다. 인덱싱은 비디오당, 인덱스 유형당 한 번만 수행하면 되는 작업입니다.

## 인덱싱 (Indexing)

### 음성 인덱스 (Spoken Word Index)

의미 기반 검색 및 키워드 검색을 위해 비디오의 음성 스크립트 콘텐츠를 인덱싱합니다:

```python
video = coll.get_video(video_id)

# force=True를 설정하면 인덱싱 작업의 멱등성(idempotent)이 보장됩니다. — 이미 인덱싱된 경우 건너뜁니다.
video.index_spoken_words(force=True)
```

이 작업은 오디오 트랙의 스크립트를 추출하고 음성 콘텐츠에 대해 검색 가능한 인덱스를 구축합니다. 의미 기반 검색(Semantic search)과 키워드 검색(Keyword search)에 필수적인 단계입니다.

**파라미터:**

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `language_code` | `str\|None` | `None` | 비디오의 언어 코드 |
| `segmentation_type` | `SegmentationType` | `SegmentationType.sentence` | 세그먼트 생성 방식 (`sentence` 또는 `llm`) |
| `force` | `bool` | `False` | `True`로 설정하면 이미 인덱싱된 경우 건너뜁니다 ("already exists" 에러 방지) |
| `callback_url` | `str\|None` | `None` | 비동기 알림을 위한 웹훅 URL |

### 장면 인덱스 (Scene Index)

장면에 대한 AI 설명을 생성하여 시각적 콘텐츠를 인덱싱합니다. 음성 인덱싱과 마찬가지로, 장면 인덱스가 이미 존재하면 에러를 발생시킵니다. 에러 메시지에서 기존의 `scene_index_id`를 추출하여 사용하십시오.

```python
import re
from videodb import SceneExtractionType

try:
    scene_index_id = video.index_scenes(
        extraction_type=SceneExtractionType.shot_based,
        prompt="이 장면의 시각적 콘텐츠, 객체, 행동 및 배경을 설명하십시오.",
    )
except Exception as e:
    match = re.search(r"id\s+([a-f0-9]+)", str(e))
    if match:
        scene_index_id = match.group(1)
    else:
        raise
```

**추출 유형 (Extraction types):**

| 유형 | 설명 | 권장 용도 |
|------|-------------|----------|
| `SceneExtractionType.shot_based` | 시각적 샷 경계를 기준으로 분할 | 일반적인 목적, 액션 위주의 콘텐츠 |
| `SceneExtractionType.time_based` | 고정된 시간 간격으로 분할 | 균일한 샘플링, 정적인 긴 콘텐츠 |
| `SceneExtractionType.transcript` | 스크립트 세그먼트를 기준으로 분할 | 대화 중심의 장면 구분 필요 시 |

**`time_based` 사용 시 파라미터 예시:**

```python
video.index_scenes(
    extraction_type=SceneExtractionType.time_based,
    extraction_config={"time": 5, "select_frames": ["first", "last"]},
    prompt="이 장면에서 무슨 일이 일어나고 있는지 설명하십시오.",
)
```

## 검색 유형 (Search Types)

### 의미 기반 검색 (Semantic Search)

음성 콘텐츠와 일치하는 자연어 쿼리를 검색합니다:

```python
from videodb import SearchType

results = video.search(
    query="머신러닝의 장점을 설명하는 구간",
    search_type=SearchType.semantic,
)
```

음성 콘텐츠가 쿼리와 의미적으로 일치하는 세그먼트들을 순위별로 반환합니다.

### 키워드 검색 (Keyword Search)

추출된 스크립트 내에서 정확한 용어 일치를 검색합니다:

```python
results = video.search(
    query="인공지능",
    search_type=SearchType.keyword,
)
```

정확한 키워드나 어구가 포함된 세그먼트를 반환합니다.

### 장면 검색 (Scene Search)

인덱싱된 장면 설명과 일치하는 시각적 콘텐츠 쿼리를 검색합니다. 사전에 `index_scenes()` 호출이 필요합니다.

`index_scenes()`는 `scene_index_id`를 반환합니다. 하나의 비디오에 여러 장면 인덱스가 있을 수 있으므로, 특정 인덱스를 대상으로 검색하려면 `video.search()`에 이 ID를 전달하십시오:

```python
from videodb import SearchType, IndexType
from videodb.exceptions import InvalidRequestError

# 장면 인덱스에 대해 의미 기반 검색 수행.
# 관련성이 낮은 노이즈를 필터링하기 위해 score_threshold 사용 (권장: 0.3 이상)
try:
    results = video.search(
        query="칠판에 글씨를 쓰는 사람",
        search_type=SearchType.semantic,
        index_type=IndexType.scene,
        scene_index_id=scene_index_id,
        score_threshold=0.3,
    )
    shots = results.get_shots()
except InvalidRequestError as e:
    if "No results found" in str(e):
        shots = []
    else:
        raise
```

**주의 사항:**

- `index_type=IndexType.scene`과 함께 `SearchType.semantic`을 사용하십시오. 이는 가장 신뢰할 수 있는 조합이며 모든 플랜에서 작동합니다.
- `SearchType.scene` 옵션도 존재하지만, 모든 플랜(예: 무료 티어)에서 사용 가능하지 않을 수 있습니다. 가급적 `IndexType.scene`과 `SearchType.semantic` 조합을 사용하십시오.
- `scene_index_id` 파라미터는 선택 사항입니다. 생략할 경우 비디오의 모든 장면 인덱스를 대상으로 검색합니다. 특정 인덱스를 지정하려면 해당 ID를 전달하십시오.
- 하나의 비디오에 대해 서로 다른 프롬프트나 추출 유형을 사용하여 여러 장면 인덱스를 생성하고, `scene_index_id`를 통해 각각 독립적으로 검색할 수 있습니다.

### 메타데이터 필터링을 포함한 장면 검색

커스텀 메타데이터와 함께 장면을 인덱싱한 경우, 의미 기반 검색과 메타데이터 필터를 결합할 수 있습니다:

```python
from videodb import SearchType, IndexType

results = video.search(
    query="긴박한 추격 장면",
    search_type=SearchType.semantic,
    index_type=IndexType.scene,
    scene_index_id=scene_index_id,
    filter=[{"camera_view": "road_ahead"}, {"action_type": "chasing"}],
)
```

커스텀 메타데이터 인덱싱 및 필터링 검색에 대한 전체 예시는 [scene_level_metadata_indexing 쿡북](https://github.com/video-db/videodb-cookbook/blob/main/quickstart/scene_level_metadata_indexing.ipynb)을 참조하십시오.

## 결과 활용하기

### 샷(Shots) 가져오기

개별 결과 세그먼트에 접근합니다:

```python
results = video.search("검색어")

for shot in results.get_shots():
    print(f"비디오 ID: {shot.video_id}")
    print(f"시작 시간: {shot.start:.2f}s")
    print(f"종료 시간: {shot.end:.2f}s")
    print(f"텍스트: {shot.text}")
    print("---")
```

### 컴파일된 결과 재생하기

모든 매칭된 세그먼트를 하나의 컴파일된 비디오로 스트리밍합니다:

```python
results = video.search("검색어")
stream_url = results.compile()
results.play()  # 브라우저에서 컴파일된 스트림 열기
```

### 클립 추출하기

특정 결과 세그먼트를 다운로드하거나 스트리밍합니다:

```python
for shot in results.get_shots():
    stream_url = shot.generate_stream()
    print(f"클립 URL: {stream_url}")
```

## 컬렉션 전체 검색 (Cross-Collection Search)

컬렉션 내의 모든 비디오를 대상으로 검색합니다:

```python
coll = conn.get_collection()

# 컬렉션의 모든 비디오 검색
results = coll.search(
    query="제품 데모",
    search_type=SearchType.semantic,
)

for shot in results.get_shots():
    print(f"비디오 ID: {shot.video_id} [{shot.start:.1f}s - {shot.end:.1f}s]")
```

> **참고:** 컬렉션 수준 검색은 오직 `SearchType.semantic`만 지원합니다. `coll.search()`에서 `SearchType.keyword`나 `SearchType.scene`을 사용하면 `NotImplementedError`가 발생합니다. 키워드 또는 장면 검색이 필요한 경우 각 비디오 에 대해 `video.search()`를 사용하십시오.

## 검색 + 컴파일 워크플로우

인덱싱, 검색, 그리고 매칭된 세그먼트들을 하나의 재생 가능한 스트림으로 컴파일하는 과정입니다:

```python
video.index_spoken_words(force=True)
results = video.search(query="검색어", search_type=SearchType.semantic)
stream_url = results.compile()
print(stream_url)
```

## 팁

- **한 번 인덱싱하고 여러 번 검색**: 인덱싱은 비용이 많이 드는 작업이지만, 한 번 완료되면 검색은 매우 빠릅니다.
- **인덱스 유형 조합**: 음성 단어와 장면 인덱싱을 모두 수행하여 동일한 비디오에 대해 모든 검색 유형을 활성화하십시오.
- **쿼리 정교화**: 의미 기반 검색은 단일 키워드보다 묘사가 풍부한 자연어 구문을 사용할 때 가장 잘 작동합니다.
- **정밀한 검색을 위해 키워드 검색 사용**: 정확한 용어 일치가 필요한 경우, 키워드 검색을 통해 의미적 편차(Semantic drift)를 방지할 수 있습니다.
- **"No results found" 처리**: 검색 결과가 없으면 `video.search()`는 `InvalidRequestError`를 발생시킵니다. 항상 검색 호출을 try/except 문으로 감싸고 `"No results found"`를 빈 결과 집합으로 처리하십시오.
- **장면 검색 노이즈 필터링**: 의미 기반 장면 검색은 모호한 쿼리에 대해 관련성이 낮은 결과를 반환할 수 있습니다. `score_threshold=0.3` (또는 그 이상)을 사용하여 노이즈를 제거하십시오.
- **멱등성 인덱싱**: 안전하게 재인덱싱하려면 `index_spoken_words(force=True)`를 사용하십시오. `index_scenes()`에는 `force` 파라미터가 없으므로, try/except 문으로 감싸고 `re.search(r"id\s+([a-f0-9]+)", str(e))`를 사용하여 에러 메시지에서 기존 `scene_index_id`를 추출하십시오.
