# 스트리밍 및 재생 (Streaming & Playback)

VideoDB는 요청 시점에 즉시 스트림을 생성하여 표준 비디오 플레이어에서 바로 감상할 수 있는 HLS 호환 URL을 반환합니다. 별도의 렌더링 시간이나 내보내기(Export) 대기 시간이 필요하지 않으며, 편집, 검색 및 구성된 결과물을 즉시 스트리밍할 수 있습니다.

## 전제 조건

스트림을 생성하려면 비디오가 먼저 컬렉션에 **업로드되어야 합니다.** 검색 기반 스트리밍의 경우, 해당 비디오가 **인덱싱**(음성 및/또는 장면)되어 있어야 합니다. 인덱싱에 대한 자세한 내용은 [search.md](search.md)를 참조하십시오.

## 핵심 개념

### 스트림 생성 (Stream Generation)

VideoDB의 모든 비디오, 검색 결과 및 타임라인은 **스트림 URL**을 생성할 수 있습니다. 이 URL은 요청 시점에 컴파일되는 HLS(HTTP Live Streaming) 매니페스트를 가리킵니다.

```python
# 비디오로부터 생성
stream_url = video.generate_stream()

# 타임라인으로부터 생성
stream_url = timeline.generate_stream()

# 검색 결과로부터 생성
stream_url = results.compile()
```

## 단일 비디오 스트리밍

### 기본 재생

```python
import videodb

conn = videodb.connect()
coll = conn.get_collection()
video = coll.get_video("your-video-id")

# 스트림 URL 생성
stream_url = video.generate_stream()
print(f"스트림 URL: {stream_url}")

# 기본 브라우저에서 열기
video.play()
```

### 자막 포함 스트리밍

```python
# 먼저 인덱싱 후 자막 추가
video.index_spoken_words(force=True)
stream_url = video.add_subtitle()

# 반환된 URL에는 이미 자막이 포함되어 있음
print(f"자막이 포함된 스트림: {stream_url}")
```

### 특정 구간 스트리밍

타임스탬프 구간을 포함한 타임라인 목록을 전달하여 특정 부분만 스트리밍할 수 있습니다:

```python
# 10-30초 및 60-90초 구간만 스트리밍
stream_url = video.generate_stream(timeline=[(10, 30), (60, 90)])
print(f"구간 스트림 URL: {stream_url}")
```

## 타임라인 구성 요소 스트리밍

여러 에셋을 조합한 결과물을 실시간으로 스트리밍합니다:

```python
import videodb
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, AudioAsset, ImageAsset, TextAsset, TextStyle

conn = videodb.connect()
coll = conn.get_collection()

video = coll.get_video(video_id)
music = coll.get_audio(music_id)

timeline = Timeline(conn)

# 메인 비디오 콘텐츠
timeline.add_inline(VideoAsset(asset_id=video.id))

# 배경 음악 오버레이 (0초부터 시작)
timeline.add_overlay(0, AudioAsset(asset_id=music.id))

# 시작 부분에 텍스트 오버레이 추가
timeline.add_overlay(0, TextAsset(
    text="라이브 데모",
    duration=3,
    style=TextStyle(fontsize=48, fontcolor="white", boxcolor="#000000"),
))

# 구성된 스트림 생성
stream_url = timeline.generate_stream()
print(f"조합된 스트림 URL: {stream_url}")
```

**중요:** `add_inline()`은 `VideoAsset`만 허용합니다. `AudioAsset`, `ImageAsset`, `TextAsset`은 `add_overlay()`를 사용하십시오.

상세한 타임라인 편집 방법은 [editor.md](editor.md)를 참조하십시오.

## 검색 결과 스트리밍

검색 결과에서 일치하는 모든 세그먼트를 하나의 스트림으로 컴파일합니다:

```python
from videodb import SearchType
from videodb.exceptions import InvalidRequestError

video.index_spoken_words(force=True)
try:
    results = video.search("주요 발표", search_type=SearchType.semantic)

    # 매칭된 모든 샷을 하나의 스트림으로 컴파일
    stream_url = results.compile()
    print(f"검색 결과 스트림 URL: {stream_url}")

    # 또는 즉시 재생
    results.play()
except InvalidRequestError as exc:
    if "No results found" in str(exc):
        print("일치하는 발표 세그먼트를 찾지 못했습니다.")
    else:
        raise
```

### 개별 검색 결과 항목 스트리밍

```python
from videodb.exceptions import InvalidRequestError

try:
    results = video.search("제품 데모", search_type=SearchType.semantic)
    for i, shot in enumerate(results.get_shots()):
        stream_url = shot.generate_stream()
        print(f"결과 {i+1} [{shot.start:.1f}s-{shot.end:.1f}s]: {stream_url}")
except InvalidRequestError as exc:
    if "No results found" in str(exc):
        print("쿼리와 일치하는 제품 데모 세그먼트가 없습니다.")
    else:
        raise
```

## 오디오 재생

오디오 콘텐츠에 대한 서명된 재생 URL을 가져옵니다:

```python
audio = coll.get_audio(audio_id)
playback_url = audio.generate_url()
print(f"오디오 URL: {playback_url}")
```

## 전체 워크플로우 예시

### 검색 후 스트리밍 파이프라인 (Search-to-Stream)

검색, 타임라인 구성, 스트리밍을 하나의 워크플로우로 결합합니다:

```python
import videodb
from videodb import SearchType
from videodb.exceptions import InvalidRequestError
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, TextAsset, TextStyle

conn = videodb.connect()
coll = conn.get_collection()
video = coll.get_video("your-video-id")

video.index_spoken_words(force=True)

# 주요 장면 검색
queries = ["introduction", "main demo", "Q&A"]
timeline = Timeline(conn)
timeline_offset = 0.0

for query in queries:
    try:
        results = video.search(query, search_type=SearchType.semantic)
        shots = results.get_shots()
    except InvalidRequestError as exc:
        if "No results found" in str(exc):
            shots = []
        else:
            raise

    if not shots:
        continue

    # 컴파일된 타임라인에서 각 섹션이 시작되는 지점에 라벨 추가
    timeline.add_overlay(timeline_offset, TextAsset(
        text=query.title(),
        duration=2,
        style=TextStyle(fontsize=36, fontcolor="white", boxcolor="#222222"),
    ))

    for shot in shots:
        timeline.add_inline(
            VideoAsset(asset_id=shot.video_id, start=shot.start, end=shot.end)
        )
        timeline_offset += shot.end - shot.start

stream_url = timeline.generate_stream()
print(f"동적 컴파일 스트림: {stream_url}")
```

### 멀티 비디오 스트림

서로 다른 비디오의 클립들을 하나의 스트림으로 결합합니다:

```python
import videodb
from videodb.timeline import Timeline
from videodb.asset import VideoAsset

conn = videodb.connect()
coll = conn.get_collection()

video_clips = [
    {"id": "vid_001", "start": 0, "end": 15},
    {"id": "vid_002", "start": 10, "end": 30},
    {"id": "vid_003", "start": 5, "end": 25},
]

timeline = Timeline(conn)
for clip in video_clips:
    timeline.add_inline(
        VideoAsset(asset_id=clip["id"], start=clip["start"], end=clip["end"])
    )

stream_url = timeline.generate_stream()
print(f"멀티 비디오 스트림 URL: {stream_url}")
```

### 조건부 스트림 조립

검색 결과 유무에 따라 동적으로 스트림을 구축합니다:

```python
import videodb
from videodb import SearchType
from videodb.exceptions import InvalidRequestError
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, TextAsset, TextStyle

conn = videodb.connect()
coll = conn.get_collection()
video = coll.get_video("your-video-id")

video.index_spoken_words(force=True)

timeline = Timeline(conn)

# 특정 콘텐츠 검색 시도; 결과가 없으면 전체 비디오로 대체
topics = ["opening remarks", "technical deep dive", "closing"]

found_any = False
timeline_offset = 0.0
for topic in topics:
    try:
        results = video.search(topic, search_type=SearchType.semantic)
        shots = results.get_shots()
    except InvalidRequestError as exc:
        if "No results found" in str(exc):
            shots = []
        else:
            raise

    if shots:
        found_any = True
        timeline.add_overlay(timeline_offset, TextAsset(
            text=topic.title(),
            duration=2,
            style=TextStyle(fontsize=32, fontcolor="white", boxcolor="#1a1a2e"),
        ))
        for shot in shots:
            timeline.add_inline(
                VideoAsset(asset_id=shot.video_id, start=shot.start, end=shot.end)
            )
            timeline_offset += shot.end - shot.start

if found_any:
    stream_url = timeline.generate_stream()
    print(f"선별된 스트림: {stream_url}")
else:
    # 검색 결과가 하나도 없는 경우 전체 비디오 스트리밍으로 대체
    stream_url = video.generate_stream()
    print(f"전체 비디오 스트림: {stream_url}")
```

### 라이브 이벤트 리캡 (Recap)

이벤트 녹화본을 여러 섹션으로 구성된 스트리밍용 리캡 영상으로 처리합니다:

```python
import videodb
from videodb import SearchType
from videodb.exceptions import InvalidRequestError
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, AudioAsset, ImageAsset, TextAsset, TextStyle

conn = videodb.connect()
coll = conn.get_collection()

# 이벤트 녹화본 업로드
event = coll.upload(url="https://example.com/event-recording.mp4")
event.index_spoken_words(force=True)

# 배경 음악 생성
music = coll.generate_music(
    prompt="경쾌한 기업용 배경 음악",
    duration=120,
)

# 타이틀 이미지 생성
title_img = coll.generate_image(
    prompt="현대적인 이벤트 리캡 타이틀 카드, 어두운 배경, 전문적인 느낌",
    aspect_ratio="16:9",
)

# 리캡 타임라인 구축
timeline = Timeline(conn)
timeline_offset = 0.0

# 검색을 통해 메인 비디오 세그먼트 추출
try:
    keynote = event.search("keynote announcement", search_type=SearchType.semantic)
    keynote_shots = keynote.get_shots()[:5]
except InvalidRequestError as exc:
    if "No results found" in str(exc):
        keynote_shots = []
    else:
        raise
if keynote_shots:
    keynote_start = timeline_offset
    for shot in keynote_shots:
        timeline.add_inline(
            VideoAsset(asset_id=shot.video_id, start=shot.start, end=shot.end)
        )
        timeline_offset += shot.end - shot.start
else:
    keynote_start = None

try:
    demo = event.search("product demo", search_type=SearchType.semantic)
    demo_shots = demo.get_shots()[:5]
except InvalidRequestError as exc:
    if "No results found" in str(exc):
        demo_shots = []
    else:
        raise
if demo_shots:
    demo_start = timeline_offset
    for shot in demo_shots:
        timeline.add_inline(
            VideoAsset(asset_id=shot.video_id, start=shot.start, end=shot.end)
        )
        timeline_offset += shot.end - shot.start
else:
    demo_start = None

# 타이틀 카드 이미지 오버레이 추가
timeline.add_overlay(0, ImageAsset(
    asset_id=title_img.id, width=100, height=100, x=80, y=20, duration=5
))

# 타임라인의 올바른 오프셋 지점에 섹션 라벨 오버레이 추가
if keynote_start is not None:
    timeline.add_overlay(max(5, keynote_start), TextAsset(
        text="기조연설 하이라이트",
        duration=3,
        style=TextStyle(fontsize=40, fontcolor="white", boxcolor="#0d1117"),
    ))
if demo_start is not None:
    timeline.add_overlay(max(5, demo_start), TextAsset(
        text="데모 하이라이트",
        duration=3,
        style=TextStyle(fontsize=36, fontcolor="white", boxcolor="#0d1117"),
    ))

# 배경 음악 오버레이 추가
timeline.add_overlay(0, AudioAsset(
    asset_id=music.id, fade_in_duration=3
))

# 최종 리캡 영상 스트리밍
stream_url = timeline.generate_stream()
print(f"이벤트 리캡 URL: {stream_url}")
```

---

## 팁

- **HLS 호환성**: 스트림 URL은 HLS 매니페스트(`.m3u8`)를 반환합니다. Safari에서는 기본적으로 작동하며, 다른 브라우저에서는 hls.js 또는 유사한 라이브러리를 통해 재생할 수 있습니다.
- **온디맨드 컴파일**: 스트림은 요청 시 서버 측에서 컴파일됩니다. 첫 재생 시 약간의 컴파일 지연이 발생할 수 있으나, 동일한 작업에 대한 이후 재생은 캐시됩니다.
- **캐싱**: 파라미터 없이 `video.generate_stream()`을 다시 호출하면 재컴파일 없이 기존에 캐시된 스트림 URL을 반환합니다.
- **구간 스트리밍**: `video.generate_stream(timeline=[(start, end)])`은 전체 `Timeline` 객체를 생성하지 않고 특정 클립만 스트리밍하는 가장 빠른 방법입니다.
- **인라인 vs 오버레이**: `add_inline()`은 `VideoAsset`만 허용하며 메인 트랙에 순차적으로 배치합니다. `add_overlay()`는 `AudioAsset`, `ImageAsset`, `TextAsset`을 허용하며 지정된 시작 시간에 레이어로 겹쳐서 배치합니다.
- **TextStyle 기본값**: `TextStyle`의 기본값은 `font='Sans'`, `fontcolor='black'`입니다. 텍스트 배경색을 지정할 때는 `bgcolor` 대신 `boxcolor`를 사용하십시오.
- **생성 기능과 결합**: `coll.generate_music(prompt, duration)` 및 `coll.generate_image(prompt, aspect_ratio)`를 사용하여 타임라인 구성에 필요한 에셋을 직접 생성해 보십시오.
- **재생**: `.play()` 메서드는 시스템 기본 브라우저에서 스트림 URL을 엽니다. 프로그래밍적인 용도로는 URL 문자열을 직접 사용하십시오.
