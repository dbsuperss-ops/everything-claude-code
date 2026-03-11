# 타임라인 편집 가이드 (Timeline Editing Guide)

VideoDB는 비파괴 방식의 타임라인 에디터를 제공합니다. 이를 통해 여러 에셋을 조합하고, 텍스 및 이미지 오버레이를 추가하며, 오디오 트랙을 믹싱하고, 클립을 자르는 작업을 수행할 수 있습니다. 이 모든 과정은 재인코딩이나 로컬 도구 없이 서버 측에서 이루어집니다. 클립 자르기, 클립 결합, 비디오에 오디오/음악 겹치기, 자막 추가, 텍스트 또는 이미지 레이어링 등에 이 기능을 활용하십시오.

## 전제 조건

비디오, 오디오 및 이미지는 타임라인 에셋으로 사용하기 전에 먼저 컬렉션에 **업로드되어야 합니다.** 자막(Caption) 오버레이의 경우, 해당 비디오가 **음성 인덱싱(indexed for spoken words)**이 완료된 상태여야 합니다.

## 핵심 개념

### 타임라인 (Timeline)

`Timeline`은 가상의 편집 레이어입니다. 에셋은 타임라인에 **인라인(inline)** 방식으로(기본 트랙에 순차적으로 배치) 또는 **오버레이(overlay)** 방식으로(특정 타임스탬프에 레이어로 배치) 추가됩니다. 원본 미디어를 변형하지 않으며, 최종 스트림은 요청 시점에 컴파일됩니다.

```python
from videodb.timeline import Timeline

timeline = Timeline(conn)
```

### 에셋 (Assets)

타임라인 상의 모든 요소는 **에셋**입니다. VideoDB는 다섯 가지 에셋 유형을 제공합니다:

| 에셋 | 임포트(Import) | 주요 용도 |
|-------|--------|-------------|
| `VideoAsset` | `from videodb.asset import VideoAsset` | 비디오 클립 (자르기, 순차 배치) |
| `AudioAsset` | `from videodb.asset import AudioAsset` | 음악, 효과음, 나레이션 |
| `ImageAsset` | `from videodb.asset import ImageAsset` | 로고, 썸네일, 오버레이 이미지 |
| `TextAsset` | `from videodb.asset import TextAsset, TextStyle` | 제목, 캡션, 하단 자막(Lower-thirds) |
| `CaptionAsset` | `from videodb.editor import CaptionAsset` | 자동 렌더링 자막 (에디터 API 전용) |

## 타임라인 구축하기

### 비디오 클립을 인라인으로 추가

인라인 에셋은 메인 비디오 트랙에서 차례대로 재생됩니다. `add_inline` 메서드는 `VideoAsset`만 허용합니다:

```python
from videodb.asset import VideoAsset

video_a = coll.get_video(video_id_a)
video_b = coll.get_video(video_id_b)

timeline = Timeline(conn)
timeline.add_inline(VideoAsset(asset_id=video_a.id))
timeline.add_inline(VideoAsset(asset_id=video_b.id))

stream_url = timeline.generate_stream()
```

### 자르기 / 부분 클립 생성

부분적인 구간만 추출하려면 `VideoAsset`의 `start`와 `end`를 사용하십시오:

```python
# 원본 비디오에서 10초부터 30초 구간만 추출
clip = VideoAsset(asset_id=video.id, start=10, end=30)
timeline.add_inline(clip)
```

### VideoAsset 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `asset_id` | `str` | 필수 | 비디오 미디어 ID |
| `start` | `float` | `0` | 자르기 시작 시간 (초) |
| `end` | `float\|None` | `None` | 자르기 종료 시간 (초, `None`은 끝까지) |

> **경고:** SDK는 음수 타임스탬프를 검증하지 않습니다. `start=-5`를 전달하면 오류 없이 받아들여지지만, 결과 스트림이 깨지거나 예상치 못한 동작이 발생할 수 있습니다. `VideoAsset`을 생성하기 전에 항상 `start >= 0`, `start < end`, `end <= video.length` 조건을 확인하십시오.

## 텍스트 오버레이

타임라인의 어느 지점에서든 제목, 하단 자막 또는 캡션을 추가할 수 있습니다:

```python
from videodb.asset import TextAsset, TextStyle

title = TextAsset(
    text="데모에 오신 것을 환영합니다",
    duration=5,
    style=TextStyle(
        fontsize=36,
        fontcolor="white",
        boxcolor="black",
        alpha=0.8,
        font="Sans",
    ),
)

# 타임라인 시작점(t=0)에 제목 오버레이 추가
timeline.add_overlay(0, title)
```

### TextStyle 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `fontsize` | `int` | `24` | 픽셀 단위 폰트 크기 |
| `fontcolor` | `str` | `"black"` | CSS 색상 이름 또는 헥사(hex) 코드 |
| `fontcolor_expr` | `str` | `""` | 동적 폰트 색상 표현식 |
| `alpha` | `float` | `1.0` | 텍스트 불투명도 (0.0–1.0) |
| `font` | `str` | `"Sans"` | 폰트 패밀리 |
| `box` | `bool` | `True` | 배경 박스 사용 여부 |
| `boxcolor` | `str` | `"white"` | 배경 박스 색상 |
| `boxborderw` | `str` | `"10"` | 박스 테두리 너비 |
| `boxw` | `int` | `0` | 박스 너비 강제 지정 |
| `boxh` | `int` | `0` | 박스 높이 강제 지정 |
| `line_spacing` | `int` | `0` | 줄 간격 |
| `text_align` | `str` | `"T"` | 박스 내 텍스트 정렬 방식 |
| `y_align` | `str` | `"text"` | 수직 정렬 기준 |
| `borderw` | `int` | `0` | 텍스트 테두리 너비 |
| `bordercolor` | `str` | `"black"` | 텍스트 테두리 색상 |
| `expansion` | `str` | `"normal"` | 텍스트 확장 모드 |
| `basetime` | `int` | `0` | 시간 기반 표현식을 위한 기준 시간 |
| `fix_bounds` | `bool` | `False` | 텍스트 경계 고정 여부 |
| `text_shaping` | `bool` | `True` | 텍스트 셰이핑(shaping) 활성화 여부 |
| `shadowcolor` | `str` | `"black"` | 그림자 색상 |
| `shadowx` | `int` | `0` | 그림자 가로 오프셋 |
| `shadowy` | `int` | `0` | 그림자 세로 오프셋 |
| `tabsize` | `int` | `4` | 탭 크기 (공백 개수) |
| `x` | `str` | `"(main_w-text_w)/2"` | 가로 위치 표현식 |
| `y` | `str` | `"(main_h-text_h)/2"` | 세로 위치 표현식 |

## 오디오 오버레이

비디오 트랙 위에 배경 음악, 효과음 또는 나레이션을 겹칠 수 있습니다:

```python
from videodb.asset import AudioAsset

music = coll.get_audio(music_id)

audio_layer = AudioAsset(
    asset_id=music.id,
    disable_other_tracks=False,
    fade_in_duration=2,
    fade_out_duration=2,
)

# 비디오 트랙 위에 t=0 시점부터 음악 오버레이 추가
timeline.add_overlay(0, audio_layer)
```

### AudioAsset 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `asset_id` | `str` | 필수 | 오디오 미디어 ID |
| `start` | `float` | `0` | 자르기 시작 시간 (초) |
| `end` | `float\|None` | `None` | 자르기 종료 시간 (초, `None`은 끝까지) |
| `disable_other_tracks` | `bool` | `True` | True일 경우 다른 오디오 트랙을 음소거 |
| `fade_in_duration` | `float` | `0` | 페이드인 시간 (초, 최대 5) |
| `fade_out_duration` | `float` | `0` | 페이드아웃 시간 (초, 최대 5) |

## 이미지 오버레이

로고, 워터마크 또는 생성된 이미지를 오버레이로 추가합니다:

```python
from videodb.asset import ImageAsset

logo = coll.get_image(logo_id)

logo_overlay = ImageAsset(
    asset_id=logo.id,
    duration=10,
    width=120,
    height=60,
    x=20,
    y=20,
)

timeline.add_overlay(0, logo_overlay)
```

### ImageAsset 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `asset_id` | `str` | 필수 | 이미지 미디어 ID |
| `width` | `int\|str` | `100` | 표시 너비 |
| `height` | `int\|str` | `100` | 표시 높이 |
| `x` | `int` | `80` | 가로 위치 (왼쪽으로부터의 픽셀 거리) |
| `y` | `int` | `20` | 세로 위치 (위쪽으로부터의 픽셀 거리) |
| `duration` | `float\|None` | `None` | 표시 시간 (초) |

## 자막(Caption) 오버레이

비디오에 자막을 추가하는 두 가지 방법이 있습니다.

### 방법 1: 자막 워크플로우 (가장 간단한 방법)

`video.add_subtitle()`을 사용하여 비디오 스트림에 직접 자막을 인코딩(Burning)합니다. 이 방식은 내부적으로 `videodb.timeline.Timeline`을 사용합니다:

```python
from videodb import SubtitleStyle

# 먼저 음성 인덱싱이 완료되어야 함 (force=True를 사용하면 이미 완료된 경우 건너뜀)
video.index_spoken_words(force=True)

# 기본 스타일로 자막 추가
stream_url = video.add_subtitle()

# 또는 자막 스타일 커스텀 지정
stream_url = video.add_subtitle(style=SubtitleStyle(
    font_name="Arial",
    font_size=22,
    primary_colour="&H00FFFFFF",
    bold=True,
))
```

### 방법 2: 에디터(Editor) API (고급 기능)

에디터 API(`videodb.editor`)는 `CaptionAsset`, `Clip`, `Track`, 그리고 고유의 `Timeline`을 사용하는 트랙 기반 구성 시스템을 제공합니다. 이는 위에서 사용한 `videodb.timeline.Timeline`과는 별개의 API입니다.

```python
from videodb.editor import (
    CaptionAsset,
    Clip,
    Track,
    Timeline as EditorTimeline,
    FontStyling,
    BorderAndShadow,
    Positioning,
    CaptionAnimation,
)

# 먼저 음성 인덱싱이 완료되어야 함 (force=True를 사용하면 이미 완료된 경우 건너뜀)
video.index_spoken_words(force=True)

# 자막 에셋 생성
caption = CaptionAsset(
    src="auto",
    font=FontStyling(name="Clear Sans", size=30),
    primary_color="&H00FFFFFF",
    back_color="&H00000000",
    border=BorderAndShadow(outline=1),
    position=Positioning(margin_v=30),
    animation=CaptionAnimation.box_highlight,
)

# 트랙과 클립을 사용하여 에디터 타임라인 구축
editor_tl = EditorTimeline(conn)
track = Track()
track.add_clip(start=0, clip=Clip(asset=caption, duration=video.length))
editor_tl.add_track(track)
stream_url = editor_tl.generate_stream()
```

### CaptionAsset 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `src` | `str` | `"auto"` | 자막 소스 (`"auto"` 또는 base64로 인코딩된 ASS 문자열) |
| `font` | `FontStyling\|None` | `FontStyling()` | 폰트 스타일 (이름, 크기, 굵게, 기울임 등) |
| `primary_color` | `str` | `"&H00FFFFFF"` | 기본 텍스트 색상 (ASS 형식) |
| `secondary_color` | `str` | `"&H000000FF"` | 보조 텍스트 색상 (ASS 형식) |
| `back_color` | `str` | `"&H00000000"` | 배경 색상 (ASS 형식) |
| `border` | `BorderAndShadow\|None` | `BorderAndShadow()` | 테두리 및 그림자 스타일 |
| `position` | `Positioning\|None` | `Positioning()` | 자막 정렬 및 여백 |
| `animation` | `CaptionAnimation\|None` | `None` | 애니메이션 효과 (예: `box_highlight`, `reveal`, `karaoke` 등) |

## 컴파일 및 스트리밍

타임라인 조립이 완료되면 이를 스트리밍 가능한 URL로 컴파일합니다. 스트림은 즉시 생성되며 렌더링을 위해 기다릴 필요가 없습니다.

```python
stream_url = timeline.generate_stream()
print(f"스트림 URL: {stream_url}")
```

더 다양한 스트리밍 옵션(세그먼트 스트림, 검색 기반 스트리밍, 오디오 재생 등)은 [streaming.md](streaming.md)를 참조하십시오.

## 전제 워크플로우 예시

### 제목 카드가 포함된 하이라이트 영상 제작

```python
import videodb
from videodb import SearchType
from videodb.exceptions import InvalidRequestError
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, TextAsset, TextStyle

conn = videodb.connect()
coll = conn.get_collection()
video = coll.get_video("your-video-id")

# 1. 주요 순간 검색
video.index_spoken_words(force=True)
try:
    results = video.search("product announcement", search_type=SearchType.semantic)
    shots = results.get_shots()
except InvalidRequestError as exc:
    if "No results found" in str(exc):
        shots = []
    else:
        raise

# 2. 타임라인 구축
timeline = Timeline(conn)

# 제목 카드 추가
title = TextAsset(
    text="제품 출시 하이라이트",
    duration=4,
    style=TextStyle(fontsize=48, fontcolor="white", boxcolor="#1a1a2e", alpha=0.95),
)
timeline.add_overlay(0, title)

# 검색된 각 클립 연결
for shot in shots:
    asset = VideoAsset(asset_id=shot.video_id, start=shot.start, end=shot.end)
    timeline.add_inline(asset)

# 3. 스트림 생성
stream_url = timeline.generate_stream()
print(f"하이라이트 영상: {stream_url}")
```

### 배경 음악과 로고 오버레이 추가

```python
import videodb
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, AudioAsset, ImageAsset

conn = videodb.connect()
coll = conn.get_collection()

main_video = coll.get_video(main_video_id)
music = coll.get_audio(music_id)
logo = coll.get_image(logo_id)

timeline = Timeline(conn)

# 메인 비디오 트랙
timeline.add_inline(VideoAsset(asset_id=main_video.id))

# 배경 음악 추가 — 비디오 원래 오디오와 믹싱하려면 disable_other_tracks=False 설정
timeline.add_overlay(
    0,
    AudioAsset(asset_id=music.id, disable_other_tracks=False, fade_in_duration=3),
)

# 처음 10초 동안 우측 상단에 로고 표시
timeline.add_overlay(
    0,
    ImageAsset(asset_id=logo.id, duration=10, x=1140, y=20, width=120, height=60),
)

stream_url = timeline.generate_stream()
print(f"최종 비디오 URL: {stream_url}")
```

### 여러 비디오를 활용한 멀티 클립 몽타주

```python
import videodb
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, TextAsset, TextStyle

conn = videodb.connect()
coll = conn.get_collection()

clips = [
    {"video_id": "vid_001", "start": 5, "end": 15, "label": "장면 1"},
    {"video_id": "vid_002", "start": 0, "end": 20, "label": "장면 2"},
    {"video_id": "vid_003", "start": 30, "end": 45, "label": "장면 3"},
]

timeline = Timeline(conn)
timeline_offset = 0.0

for clip in clips:
    # 각 클립 위에 라벨 오버레이 추가
    label = TextAsset(
        text=clip["label"],
        duration=2,
        style=TextStyle(fontsize=32, fontcolor="white", boxcolor="#333333"),
    )
    timeline.add_inline(
        VideoAsset(asset_id=clip["video_id"], start=clip["start"], end=clip["end"])
    )
    timeline.add_overlay(timeline_offset, label)
    timeline_offset += clip["end"] - clip["start"]

stream_url = timeline.generate_stream()
print(f"몽타주 영상 URL: {stream_url}")
```

## 두 가지 타임라인 API

VideoDB에는 두 가지의 독립적인 타임라인 시스템이 있습니다. 이들은 **서로 호환되지 않습니다**:

| | `videodb.timeline.Timeline` | `videodb.editor.Timeline` (에디터 API) |
|---|---|---|
| **임포트** | `from videodb.timeline import Timeline` | `from videodb.editor import Timeline as EditorTimeline` |
| **에셋** | `VideoAsset`, `AudioAsset`, `ImageAsset`, `TextAsset` | `CaptionAsset`, `Clip`, `Track` |
| **메서드** | `add_inline()`, `add_overlay()` | `Track` / `Clip`을 사용한 `add_track()` |
| **권장 용도** | 비디오 조합, 오버레이, 멀티 클립 편집 | 애니메이션 포함 자막/캡션 스타일링 |

한 API의 에셋을 다른 API에서 섞어 사용하지 마십시오. `CaptionAsset`은 에디터 API에서만 작동합니다. `VideoAsset` / `AudioAsset` / `ImageAsset` / `TextAsset`은 `videodb.timeline.Timeline`에서만 작동합니다.

## 제한 사항 및 제약

타임라인 에디터는 **비파괴 선형 구성**을 위해 설계되었습니다. 다음 작업들은 **지원되지 않습니다**:

### 불가한 작업

| 제한 사항 | 상세 내용 |
|---|---|
| **장면 전환 효과(Transitions) 없음** | 클립 간의 크로스페이드, 와이프, 디졸브 또는 전환 효과가 없습니다. 모든 컷은 장면이 즉시 바뀌는 하드 컷(Hard cut)입니다. |
| **비디오 위 비디오(PIP) 불가** | `add_inline()`은 `VideoAsset`만 허용합니다. 하나의 비디오 스트림 위에 다른 비디오 스트림을 겹칠 수 없습니다. 이미지 오버레이로 정적인 PIP 효과는 가능하지만 실시간 비디오 PIP는 불가능합니다. |
| **속도 또는 재생 제어 불가** | 슬로우 모션, 빨리 감기, 역재생 또는 타임 리매핑 기능이 없습니다. `VideoAsset`에는 `speed` 파라미터가 없습니다. |
| **크롭, 줌 또는 팬(Pan) 불가** | 비디오 프레임의 특정 영역을 자르거나, 줌 효과를 주거나, 프레임을 따라 팬 하는 기능이 없습니다. `video.reframe()`은 가로세로 비율 변환 전용입니다. |
| **비디오 필터 또는 컬러 그레이딩 불가** | 밝기, 대비, 채도, 색조 또는 색상 교정 기능이 없습니다. |
| **애니메이션 텍스트 불가** | `TextAsset`은 지정된 시간 동안 정적입니다. 페이드인/아웃, 움직임 또는 애니메이션이 없습니다. 애니메이션 캡션이 필요한 경우 에디터 API의 `CaptionAsset`을 사용하십시오. |
| **복합 텍스트 스타일 불가** | 하나의 `TextAsset`은 하나의 `TextStyle`만 가집니다. 단일 텍스트 블록 내에서 굵게, 기울임 또는 여러 색상을 섞어 쓸 수 없습니다. |
| **빈 화면 또는 단색 클립 불가** | 단색 프레임, 검은 화면 또는 자막만 나오는 독립적인 카드를 생성할 수 없습니다. 텍스트 및 이미지 오버레이를 사용하려면 인라인 트랙에 베이스가 되는 `VideoAsset`이 있어야 합니다. |
| **오디오 볼륨 개별 제어 불가** | `AudioAsset`에는 `volume` 파라미터가 없습니다. 오디오는 `disable_other_tracks`를 통해 최대 볼륨으로 출력되거나 음소거됩니다. 특정 수준으로 볼륨을 줄여 믹싱할 수 없습니다. |
| **키프레임 애니메이션 불가** | 시간에 따라 오버레이 속성을 변경할 수 없습니다 (예: 이미지를 A 지점에서 B 지점으로 이동). |

### 제약 사항

| 제약 사항 | 상세 내용 |
|---|---|
| **오디오 페이드 최대 5초** | `fade_in_duration` 및 `fade_out_duration`은 각각 최대 5초로 제한됩니다. |
| **오버레이 위치는 절대적임** | 오버레이는 타임라인 시작점으로부터의 절대 타임스탬프를 사용합니다. 인라인 클립의 순서를 바꿔도 해당 위치의 오버레이는 이동하지 않습니다. |
| **인라인 트랙은 비디오 전용임** | `add_inline()`은 `VideoAsset`만 허용합니다. 오디오, 이미지 및 텍스트는 `add_overlay()`를 사용해야 합니다. |
| **오버레이-클립 바인딩 없음** | 오버레이는 고정된 타임라인 타임스탬프에 배치됩니다. 특정 인라인 클립에 오버레이를 연결하여 함께 움직이게 하는 방법은 없습니다. |

## 팁

- **비파괴 방식**: 타임라인은 원본 미디어를 절대 수정하지 않습니다. 동일한 에셋으로 여러 개의 타임라인을 만들 수 있습니다.
- **오버레이 중첩**: 동일한 타임스탬프에 여러 오버레이를 시작할 수 있습니다. 오디오 오버레이는 서로 믹싱되며, 이미지/텍스트 오버레이는 추가된 순서대로 레이어링됩니다.
- **인라인은 VideoAsset 전용**: `add_inline()`은 `VideoAsset`만 허용합니다. `AudioAsset`, `ImageAsset`, `TextAsset`은 `add_overlay()`를 사용하십시오.
- **자르기 정밀도**: `VideoAsset` 및 `AudioAsset`의 `start`/`end` 단위는 초(seconds)입니다.
- **비디오 오디오 음소거**: 음악이나 나레이션을 겹칠 때 원본 비디오 오디오를 끄려면 `AudioAsset`에서 `disable_other_tracks=True`로 설정하십시오.
- **페이드 제한**: `AudioAsset`의 `fade_in_duration` 및 `fade_out_duration`은 최대 5초입니다.
- **생성 미디어 활용**: `coll.generate_music()`, `coll.generate_sound_effect()`, `coll.generate_voice()`, `coll.generate_image()`를 통해 즉시 타임라인 에셋으로 사용할 수 있는 미디어를 생성하십시오.
