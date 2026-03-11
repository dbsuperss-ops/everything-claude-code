# 전체 API 레퍼런스 (Complete API Reference)

VideoDB 스킬을 위한 참조 자료입니다. 사용법 가이드 및 워크플로우 선택에 대해서는 [../SKILL.md](../SKILL.md)를 먼저 확인하십시오.

## 연결 (Connection)

```python
import videodb

conn = videodb.connect(
    api_key="your-api-key",      # 또는 VIDEO_DB_API_KEY 환경 변수 설정
    base_url=None,                # 커스텀 API 엔드포인트 (선택 사항)
)
```

**반환값:** `Connection` 객체

### 연결(Connection) 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `conn.get_collection(collection_id="default")` | `Collection` | 컬렉션 가져오기 (ID가 없으면 기본 컬렉션) |
| `conn.get_collections()` | `list[Collection]` | 모든 컬렉션 목록 표시 |
| `conn.create_collection(name, description, is_public=False)` | `Collection` | 새 컬렉션 생성 |
| `conn.update_collection(id, name, description)` | `Collection` | 컬렉션 업데이트 |
| `conn.check_usage()` | `dict` | 계정 사용량 통계 확인 |
| `conn.upload(source, media_type, name, ...)` | `Video\|Audio\|Image` | 기본 컬렉션에 업로드 |
| `conn.record_meeting(meeting_url, bot_name, ...)` | `Meeting` | 회의 녹화 |
| `conn.create_capture_session(...)` | `CaptureSession` | 캡처 세션 생성 ([capture-reference.md](capture-reference.md) 참조) |
| `conn.youtube_search(query, result_threshold, duration)` | `list[dict]` | YouTube 검색 |
| `conn.transcode(source, callback_url, mode, ...)` | `str` | 비디오 트랜스코딩 (작업 ID 반환) |
| `conn.get_transcode_details(job_id)` | `dict` | 트랜스코딩 작업 상태 및 상세 내용 확인 |
| `conn.connect_websocket(collection_id)` | `WebSocketConnection` | WebSocket 연결 ([capture-reference.md](capture-reference.md) 참조) |

### 트랜스코딩 (Transcode)

커스텀 해상도, 품질 및 오디오 설정을 적용하여 URL로부터 비디오를 트랜스코딩합니다. 처리는 서버 측에서 이루어지므로 로컬 ffmpeg가 필요하지 않습니다.

```python
from videodb import TranscodeMode, VideoConfig, AudioConfig

job_id = conn.transcode(
    source="https://example.com/video.mp4",
    callback_url="https://example.com/webhook",
    mode=TranscodeMode.economy,
    video_config=VideoConfig(resolution=720, quality=23),
    audio_config=AudioConfig(mute=False),
)
```

#### transcode 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `source` | `str` | 필수 | 트랜스코딩할 비디오의 URL (다운로드 가능한 URL 권장) |
| `callback_url` | `str` | 필수 | 트랜스코딩 완료 시 콜백을 받을 URL |
| `mode` | `TranscodeMode` | `TranscodeMode.economy` | 트랜스코딩 속도: `economy` 또는 `lightning` |
| `video_config` | `VideoConfig` | `VideoConfig()` | 비디오 인코딩 설정 |
| `audio_config` | `AudioConfig` | `AudioConfig()` | 오디오 인코딩 설정 |

작업 ID(`str`)를 반환합니다. 작업 상태를 확인하려면 `conn.get_transcode_details(job_id)`를 사용하십시오.

```python
details = conn.get_transcode_details(job_id)
```

#### VideoConfig

```python
from videodb import VideoConfig, ResizeMode

config = VideoConfig(
    resolution=720,              # 대상 해상도 높이 (예: 480, 720, 1080)
    quality=23,                  # 인코딩 품질 (낮을수록 좋음, 기본값 23)
    framerate=30,                # 대상 프레임레이트
    aspect_ratio="16:9",         # 대상 가로세로 비율
    resize_mode=ResizeMode.crop, # 맞춤 방식: crop(자르기), fit(맞춤), pad(여백 추가)
)
```

| 필드 | 타입 | 기본값 | 설명 |
|-------|------|---------|-------------|
| `resolution` | `int\|None` | `None` | 픽셀 단위 대상 해상도 높이 |
| `quality` | `int` | `23` | 인코딩 품질 (낮을수록 고품질) |
| `framerate` | `int\|None` | `None` | 대상 프레임레이트 |
| `aspect_ratio` | `str\|None` | `None` | 대상 가로세로 비율 (예: `"16:9"`, `"9:16"`) |
| `resize_mode` | `str` | `ResizeMode.crop` | 크기 조정 전략: `crop`, `fit`, 또는 `pad` |

#### AudioConfig

```python
from videodb import AudioConfig

config = AudioConfig(mute=False)
```

| 필드 | 타입 | 기본값 | 설명 |
|-------|------|---------|-------------|
| `mute` | `bool` | `False` | 오디오 트랙 음소거 여부 |

## 컬렉션 (Collections)

```python
coll = conn.get_collection()
```

### 컬렉션(Collection) 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `coll.get_videos()` | `list[Video]` | 모든 비디오 목록 표시 |
| `coll.get_video(video_id)` | `Video` | 특정 비디오 가져오기 |
| `coll.get_audios()` | `list[Audio]` | 모든 오디오 목록 표시 |
| `coll.get_audio(audio_id)` | `Audio` | 특정 오디오 가져오기 |
| `coll.get_images()` | `list[Image]` | 모든 이미지 목록 표시 |
| `coll.get_image(image_id)` | `Image` | 특정 이미지 가져오기 |
| `coll.upload(url=None, file_path=None, media_type=None, name=None)` | `Video\|Audio\|Image` | 미디어 업로드 |
| `coll.search(query, search_type, index_type, score_threshold, namespace, scene_index_id, ...)` | `SearchResult` | 컬렉션 전체 검색 (의미적 검색만 가능하며, 키워드 및 장면 검색은 `NotImplementedError` 발생) |
| `coll.generate_image(prompt, aspect_ratio="1:1")` | `Image` | IT 기반 이미지 생성 |
| `coll.generate_video(prompt, duration=5)` | `Video` | AI 기반 비디오 생성 |
| `coll.generate_music(prompt, duration=5)` | `Audio` | AI 기반 음악 생성 |
| `coll.generate_sound_effect(prompt, duration=2)` | `Audio` | 효과음 생성 |
| `coll.generate_voice(text, voice_name="Default")` | `Audio` | 텍스트로부터 음성 생성(TTS) |
| `coll.generate_text(prompt, model_name="basic", response_type="text")` | `dict` | LLM 텍스트 생성 — `["output"]`을 통해 결과 확인 |
| `coll.dub_video(video_id, language_code)` | `Video` | 다른 언어로 비디오 더빙 |
| `coll.record_meeting(meeting_url, bot_name, ...)` | `Meeting` | 실시간 회의 녹화 |
| `coll.create_capture_session(...)` | `CaptureSession` | 캡처 세션 생성 ([capture-reference.md](capture-reference.md) 참조) |
| `coll.get_capture_session(...)` | `CaptureSession` | 캡처 세션 검색 ([capture-reference.md](capture-reference.md) 참조) |
| `coll.connect_rtstream(url, name, ...)` | `RTStream` | 라이브 스트림 연결 ([rtstream-reference.md](rtstream-reference.md) 참조) |
| `coll.make_public()` | `None` | 컬렉션을 공개로 전환 |
| `coll.make_private()` | `None` | 컬렉션을 비공개로 전환 |
| `coll.delete_video(video_id)` | `None` | 비디오 삭제 |
| `coll.delete_audio(audio_id)` | `None` | 오디오 삭제 |
| `coll.delete_image(image_id)` | `None` | 이미지 삭제 |
| `coll.delete()` | `None` | 컬렉션 삭제 |

### 업로드 파라미터

```python
video = coll.upload(
    url=None,            # 원격 URL (HTTP, YouTube)
    file_path=None,      # 로컬 파일 경로
    media_type=None,     # "video", "audio", 또는 "image" (생략 시 자동 감지)
    name=None,           # 미디어의 커스텀 이름
    description=None,    # 설명
    callback_url=None,   # 비동기 알림을 위한 웹훅 URL
)
```

## Video 객체

```python
video = coll.get_video(video_id)
```

### Video 속성 (Properties)

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `video.id` | `str` | 고유 비디오 ID |
| `video.collection_id` | `str` | 상위 컬렉션 ID |
| `video.name` | `str` | 비디오 이름 |
| `video.description` | `str` | 비디오 설명 |
| `video.length` | `float` | 초 단위 재생 시간 |
| `video.stream_url` | `str` | 기본 스트림 URL |
| `video.player_url` | `str` | 플레이어 임베드 URL |
| `video.thumbnail_url` | `str` | 썸네일 URL |

### Video 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `video.generate_stream(timeline=None)` | `str` | 스트림 URL 생성 (선택적으로 `[(start, end)]` 튜플 형태의 타임라인 전달 가능) |
| `video.play()` | `str` | 브라우저에서 스트림 열기, 플레이어 URL 반환 |
| `video.index_spoken_words(language_code=None, force=False)` | `None` | 검색을 위한 음성 인덱싱. 이미 인덱싱된 경우 건너뛰려면 `force=True` 사용. |
| `video.index_scenes(extraction_type, prompt, extraction_config, metadata, model_name, name, scenes, callback_url)` | `str` | 시각적 장면 인덱싱 (scene_index_id 반환) |
| `video.index_visuals(prompt, batch_config, ...)` | `str` | 시각 요소 인덱싱 (scene_index_id 반환) |
| `video.index_audio(prompt, model_name, ...)` | `str` | LLM을 사용한 오디오 인덱싱 (scene_index_id 반환) |
| `video.get_transcript(start=None, end=None)` | `list[dict]` | 타임스탬프가 포함된 스크립트 가져오기 |
| `video.get_transcript_text(start=None, end=None)` | `str` | 전체 스크립트 텍스트 가져오기 |
| `video.generate_transcript(force=None)` | `dict` | 스크립트 생성 |
| `video.translate_transcript(language, additional_notes)` | `list[dict]` | 스크립트 번역 |
| `video.search(query, search_type, index_type, filter, **kwargs)` | `SearchResult` | 비디오 내부 검색 |
| `video.add_subtitle(style=SubtitleStyle())` | `str` | 자막 추가 (스트림 URL 반환) |
| `video.generate_thumbnail(time=None)` | `str\|Image` | 썸네일 생성 |
| `video.get_thumbnails()` | `list[Image]` | 모든 썸네일 가져오기 |
| `video.extract_scenes(extraction_type, extraction_config)` | `SceneCollection` | 장면 추출 |
| `video.reframe(start, end, target, mode, callback_url)` | `Video\|None` | 비디오 가로세로 비율 재설정 |
| `video.clip(prompt, content_type, model_name)` | `str` | 프롬프트로부터 클립 생성 (스트림 URL 반환) |
| `video.insert_video(video, timestamp)` | `str` | 타임스탬프 위치에 비디오 삽입 |
| `video.download(name=None)` | `dict` | 비디오 다운로드 |
| `video.delete()` | `None` | 비디오 삭제 |

### Reframe

선택적으로 스마트 객체 추적 기능을 사용하여 비디오를 다른 가로세로 비율로 변환합니다. 처리는 서버 측에서 이루어집니다.

> **경고:** Reframe은 속도가 느린 서버 측 작업입니다. 긴 비디오의 경우 몇 분이 소요될 수 있으며 타임아웃이 발생할 수 있습니다. 항상 `start`/`end`를 사용하여 세그먼트를 작게 제한하거나, 비동기 처리를 위해 `callback_url`을 사용하십시오.

```python
from videodb import ReframeMode

# 타임아웃 방지를 위해 항상 짧은 세그먼트를 선호하십시오:
reframed = video.reframe(start=0, end=60, target="vertical", mode=ReframeMode.smart)

# 전체 비디오에 대한 비동기 재설정 (None 반환, 결과는 웹훅을 통해 수신):
video.reframe(target="vertical", callback_url="https://example.com/webhook")

# 커스텀 크기 설정
reframed = video.reframe(start=0, end=60, target={"width": 1080, "height": 1080})
```

#### reframe 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `start` | `float\|None` | `None` | 초 단위 시작 시간 (None = 처음부터) |
| `end` | `float\|None` | `None` | 초 단위 종료 시간 (None = 비디오 끝까지) |
| `target` | `str\|dict` | `"vertical"` | 프리셋 문자열 (`"vertical"`, `"square"`, `"landscape"`) 또는 `{"width": int, "height": int}` |
| `mode` | `str` | `ReframeMode.smart` | `"simple"` (중앙 자르기) 또는 `"smart"` (객체 추적) |
| `callback_url` | `str\|None` | `None` | 비동기 알림을 위한 웹훅 URL |

`callback_url`이 제공되지 않으면 `Video` 객체를 반환하고, 제공되면 `None`을 반환합니다.

## Audio 객체

```python
audio = coll.get_audio(audio_id)
```

### Audio 속성 (Properties)

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `audio.id` | `str` | 고유 오디오 ID |
| `audio.collection_id` | `str` | 상위 컬렉션 ID |
| `audio.name` | `str` | 오디오 이름 |
| `audio.length` | `float` | 초 단위 재생 시간 |

### Audio 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `audio.generate_url()` | `str` | 재생을 위한 서명된(Signed) URL 생성 |
| `audio.get_transcript(start=None, end=None)` | `list[dict]` | 타임스탬프가 포함된 스크립트 가져오기 |
| `audio.get_transcript_text(start=None, end=None)` | `str` | 전체 스크립트 텍스트 가져오기 |
| `audio.generate_transcript(force=None)` | `dict` | 스크립트 생성 |
| `audio.delete()` | `None` | 오디오 삭제 |

## Image 객체

```python
image = coll.get_image(image_id)
```

### Image 속성 (Properties)

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `image.id` | `str` | 고유 이미지 ID |
| `image.collection_id` | `str` | 상위 컬렉션 ID |
| `image.name` | `str` | 이미지 이름 |
| `image.url` | `str\|None` | 이미지 URL (생성된 이미지의 경우 `None`일 수 있으므로 대신 `generate_url()` 사용) |

### Image 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `image.generate_url()` | `str` | 서명된 URL 생성 |
| `image.delete()` | `None` | 이미지 삭제 |

## 타임라인 및 에디터 (Timeline & Editor)

### 타임라인 (Timeline)

```python
from videodb.timeline import Timeline

timeline = Timeline(conn)
```

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `timeline.add_inline(asset)` | `None` | 메인 트랙에 순차적으로 `VideoAsset` 추가 |
| `timeline.add_overlay(start, asset)` | `None` | 특정 타임스탬프에 `AudioAsset`, `ImageAsset` 또는 `TextAsset` 오버레이 추가 |
| `timeline.generate_stream()` | `str` | 컴파일 및 스트림 URL 가져오기 |

### 에셋 유형 (Asset Types)

#### VideoAsset

```python
from videodb.asset import VideoAsset

asset = VideoAsset(
    asset_id=video.id,
    start=0,              # 자르기 시작 (초)
    end=None,             # 자르기 종료 (초, None = 끝까지)
)
```

#### AudioAsset

```python
from videodb.asset import AudioAsset

asset = AudioAsset(
    asset_id=audio.id,
    start=0,
    end=None,
    disable_other_tracks=True,   # True일 경우 원래 오디오 음소거
    fade_in_duration=0,          # 초 단위 (최대 5)
    fade_out_duration=0,         # 초 단위 (최대 5)
)
```

#### ImageAsset

```python
from videodb.asset import ImageAsset

asset = ImageAsset(
    asset_id=image.id,
    duration=None,        # 표시 시간 (초)
    width=100,            # 표시 너비
    height=100,           # 표시 높이
    x=80,                 # 가로 위치 (왼쪽으로부터의 픽셀 거리)
    y=20,                 # 세로 위치 (위쪽으로부터의 픽셀 거리)
)
```

#### TextAsset

```python
from videodb.asset import TextAsset, TextStyle

asset = TextAsset(
    text="Hello World",
    duration=5,
    style=TextStyle(
        fontsize=24,
        fontcolor="black",
        boxcolor="white",       # 배경 박스 색상
        alpha=1.0,
        font="Sans",
        text_align="T",         # 박스 내 텍스트 정렬
    ),
)
```

#### CaptionAsset (Editor API)

CaptionAsset은 에디터(Editor) API에 속하며, 고유의 타임라인, 트랙 및 클립 시스템을 가지고 있습니다:

```python
from videodb.editor import CaptionAsset, FontStyling

asset = CaptionAsset(
    src="auto",                    # "auto" 또는 base64로 인코딩된 ASS 문자열
    font=FontStyling(name="Clear Sans", size=30),
    primary_color="&H00FFFFFF",
)
```

에디터 API에서의 전체 CaptionAsset 사용법은 [editor.md](editor.md#caption-overlays)를 참조하십시오.

## 비디오 검색 파라미터

```python
results = video.search(
    query="검색어",
    search_type=SearchType.semantic,       # semantic(의미), keyword(키워드), 또는 scene(장면)
    index_type=IndexType.spoken_word,      # spoken_word 또는 scene
    result_threshold=None,                 # 최대 결과 수
    score_threshold=None,                  # 최소 관련성 점수
    dynamic_score_percentage=None,         # 동적 점수 비율
    scene_index_id=None,                   # 특정 장면 인덱스 타겟팅 (**kwargs를 통해 전달)
    filter=[],                             # 장면 검색을 위한 메타데이터 필터
)
```

> **참고:** `filter`는 `video.search()`에서 명시적으로 정의된 파라미터입니다. `scene_index_id`는 `**kwargs`를 통해 API로 전달됩니다.

> **중요:** 검색 결과가 없는 경우 `video.search()`는 `"No results found"`라는 메시지와 함께 `InvalidRequestError`를 발생시킵니다. 항상 검색 호출을 try/except 문으로 감싸십시오. 장면 검색의 경우, 관련성이 낮은 노이즈를 필터링하기 위해 `score_threshold=0.3` 이상을 사용하는 것을 권장합니다.

장면 검색 시에는 `index_type=IndexType.scene`과 함께 `search_type=SearchType.semantic`을 사용하십시오. 특정 장면 인덱스를 대상으로 할 때는 `scene_index_id`를 전달하십시오. 자세한 내용은 [search.md](search.md)를 참조하십시오.

## SearchResult 객체

```python
results = video.search("쿼리", search_type=SearchType.semantic)
```

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `results.get_shots()` | `list[Shot]` | 일치하는 세그먼트 목록 가져오기 |
| `results.compile()` | `str` | 모든 샷(Shots)을 하나의 스트림 URL로 컴파일 |
| `results.play()` | `str` | 브라우저에서 컴파일된 스트림 열기 |

### Shot 속성 (Properties)

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `shot.video_id` | `str` | 원본 비디오 ID |
| `shot.video_length` | `float` | 원본 비디오 길이 |
| `shot.video_title` | `str` | 원본 비디오 제목 |
| `shot.start` | `float` | 시작 시간 (초) |
| `shot.end` | `float` | 종료 시간 (초) |
| `shot.text` | `str` | 매칭된 텍스트 내용 |
| `shot.search_score` | `float` | 검색 관련성 점수 |

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `shot.generate_stream()` | `str` | 이 특정 샷에 대한 스트리밍 실행 |
| `shot.play()` | `str` | 브라우저에서 샷 스트림 열기 |

## Meeting 객체

```python
meeting = coll.record_meeting(
    meeting_url="https://meet.google.com/...",
    bot_name="Bot",
    callback_url=None,          # 상태 업데이트를 위한 웹훅 URL
    callback_data=None,         # 콜백에 전달되는 선택적 dict
    time_zone="UTC",            # 회의 시간대
)
```

### Meeting 속성 (Properties)

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `meeting.id` | `str` | 고유 회의 ID |
| `meeting.collection_id` | `str` | 상위 컬렉션 ID |
| `meeting.status` | `str` | 현재 상태 |
| `meeting.video_id` | `str` | 녹화된 비디오 ID (완료 후 부여) |
| `meeting.bot_name` | `str` | 봇 이름 |
| `meeting.meeting_title` | `str` | 회의 제목 |
| `meeting.meeting_url` | `str` | 회의 URL |
| `meeting.speaker_timeline` | `dict` | 화자별 타임라인 데이터 |
| `meeting.is_active` | `bool` | 초기화 중이거나 처리 중인 경우 True |
| `meeting.is_completed` | `bool` | 완료된 경우 True |

### Meeting 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `meeting.refresh()` | `Meeting` | 서버로부터 데이터 갱신 |
| `meeting.wait_for_status(target_status, timeout=14400, interval=120)` | `bool` | 대상 상태에 도달할 때까지 폴링(Polling) |

## RTStream 및 캡처 (Capture)

RTStream (실시간 수집, 인덱싱, 스크립트 추출)에 대해서는 [rtstream-reference.md](rtstream-reference.md)를 참조하십시오.

캡처 세션 (데스크톱 녹화, CaptureClient, 채널)에 대해서는 [capture-reference.md](capture-reference.md)를 참조하십시오.

## 열거형(Enums) 및 상수

### SearchType

```python
from videodb import SearchType

SearchType.semantic    # 자연어 의미 검색
SearchType.keyword     # 정확한 키워드 매칭
SearchType.scene       # 시각적 장면 검색 (유료 플랜 필요 가능성 있음)
SearchType.llm         # AI 기반 검색
```

### SceneExtractionType

```python
from videodb import SceneExtractionType

SceneExtractionType.shot_based   # 자동 샷 경계 감지
SceneExtractionType.time_based   # 고정 시간 간격 추출
SceneExtractionType.transcript   # 스크립트 기반 장면 추출
```

### SubtitleStyle

```python
from videodb import SubtitleStyle

style = SubtitleStyle(
    font_name="Arial",
    font_size=18,
    primary_colour="&H00FFFFFF",
    bold=False,
    # ... 모든 옵션은 SubtitleStyle 클래스 참조
)
video.add_subtitle(style=style)
```

### SubtitleAlignment 및 SubtitleBorderStyle

```python
from videodb import SubtitleAlignment, SubtitleBorderStyle
```

### TextStyle

```python
from videodb import TextStyle
# 또는: from videodb.asset import TextStyle

style = TextStyle(
    fontsize=24,
    fontcolor="black",
    boxcolor="white",
    font="Sans",
    text_align="T",
    alpha=1.0,
)
```

### 기타 상수

```python
from videodb import (
    IndexType,          # spoken_word, scene
    MediaType,          # video, audio, image
    Segmenter,          # word, sentence, time
    SegmentationType,   # sentence, llm
    TranscodeMode,      # economy, lightning
    ResizeMode,         # crop, fit, pad
    ReframeMode,        # simple, smart
    RTStreamChannelType,
)
```

## 예외 (Exceptions)

```python
from videodb.exceptions import (
    AuthenticationError,     # API 키 누락 또는 유효하지 않음
    InvalidRequestError,     # 잘못된 파라미터 또는 형식이 어긋난 요청
    RequestTimeoutError,     # 요청 시간 초과
    SearchError,             # 검색 작업 실패 (예: 인덱스 부재 등)
    VideodbError,            # 모든 VideoDB 에러의 기본 예외 클래스
)
```

| 예외 | 주요 원인 |
|-----------|-------------|
| `AuthenticationError` | `VIDEO_DB_API_KEY` 누락 또는 유효하지 않음 |
| `InvalidRequestError` | 잘못된 URL, 지원되지 않는 형식, 잘못된 파라미터 등 |
| `RequestTimeoutError` | 서버 응답 지연 |
| `SearchError` | 인덱싱 전 검색 시도, 지원되지 않는 검색 유형 등 |
| `VideodbError` | 서버 에러, 네트워크 문제, 일반적인 실패 상황 |
