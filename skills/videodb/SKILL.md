---
name: videodb
description: 비디오 및 오디오 데이터를 보고, 이해하고, 활용하십시오. 보기(See)- 로컬 파일, URL, RTSP/라이브 피드 또는 데스크톱 라이브 녹화물로부터 수집하고, 실시간 컨텍스트 및 재생 가능한 스트림 링크를 반환합니다. 이해하기(Understand)- 프레임을 추출하고, 시각적/의미적/시간적 인덱스를 구축하며, 타임스탬프와 자동 클립을 통해 원하는 순간을 검색합니다. 활용하기(Act)- 트랜스코딩 및 정규화(코덱, fps, 해상도, 가로세로 비율), 타임라인 편집(자막, 텍스트/이미지 오버레이, 브랜딩, 오디오 오버레이, 더빙, 번역), 미디어 에셋 생성(이미지, 오디오, 비디오), 라이브 스트림이나 데스크톱 캡처 이벤트에 대한 실시간 알림 생성 등의 작업을 수행합니다.
origin: ECC
allowed-tools: Read Grep Glob Bash(python:*)
argument-hint: "[작업 설명]"
---

# VideoDB 스킬 (Skill)

**비디오, 라이브 스트림, 데스크톱 세션에 대한 인지 + 메모리 + 액션 기능을 제공합니다.**

## 사용 시기

### 데스크톱 인지 (Desktop Perception)
- **화면, 마이크 및 시스템 오디오**를 캡처하는 **데스크톱 세션**을 시작/중단할 때
- **실시간 컨텍스트**를 스트리밍하고 **에피소드 형식의 세션 메모리**를 저장할 때
- 발화 내용과 화면상의 사건에 대해 **실시간 알림/트리거**를 실행할 때
- **세션 요약**, 검색 가능한 타임라인 및 **재생 가능한 증거 링크**를 생성할 때

### 비디오 수집 + 스트림 (Video Ingest + Stream)
- **파일 또는 URL**을 수집하여 **재생 가능한 웹 스트림 링크**를 반환할 때
- 트랜스코딩/정규화: **코덱, 비트레이트, fps, 해상도, 가로세로 비율**을 변환할 때

### 인덱스 + 검색 (타임스탬프 + 증거)
- **비주얼(Visual)**, **음성(Spoken)**, **키워드(Keyword)** 인덱스를 구축할 때
- **타임스탬프**와 **재생 가능한 증거**와 함께 정확한 순간을 검색하고 반환할 때
- 검색 결과로부터 **클립**을 자동으로 생성할 때

### 타임라인 편집 + 생성
- 자막: **생성**, **번역**, **인코딩(Burn-in)**
- 오버레이: **텍스트/이미지/브랜딩**, 모션 캡션
- 오디오: **배경 음악**, **나레이션**, **더빙**
- **타임라인 작업**을 통한 프로그래밍 방식의 구성 및 내보내기

### 라이브 스트림 (RTSP) + 모니터링
- **RTSP/라이브 피드**를 연결할 때
- 모니터링 워크플로우를 위해 **실시간 시각 및 음성 이해**를 실행하고 **이벤트/알림**을 보낼 때

## 작동 방식

### 주요 입력
- 로컬 **파일 경로**, 공개 **URL** 또는 **RTSP URL**
- 데스크톱 캡처 요청: **세션 시작 / 중단 / 요약**
- 주요 작업 사양: 이해를 위한 컨텍스트 확인, 트랜스코딩 스펙, 인덱스 스펙, 검색 쿼리, 클립 범위, 타임라인 편집, 알림 규칙

### 주요 출력
- **스트림 URL**
- **타임스탬프** 및 **증거 링크**가 포함된 검색 결과
- 생성된 에셋: 자막, 오디오, 이미지, 클립
- 라이브 스트림을 위한 **이벤트/알림 페이로드**
- 데스크톱 **세션 요약** 및 메모리 항목

### Python 코드 실행

VideoDB 코드를 실행하기 전, 프로젝트 디렉토리로 이동하여 환경 변수를 로드하십시오:

```python
from dotenv import load_dotenv
load_dotenv(".env")

import videodb
conn = videodb.connect()
```

이 코드는 다음 위치에서 `VIDEO_DB_API_KEY`를 읽습니다:
1. 환경 변수 (이미 내보내기(export)된 경우)
2. 현재 디렉토리에 있는 프로젝트의 `.env` 파일

키가 없는 경우, `videodb.connect()`는 자동으로 `AuthenticationError`를 발생시킵니다.

짧은 인라인 명령으로 해결 가능한 경우에는 별도의 스크립트 파일을 작성하지 마십시오.

인라인 Python(`python -c "..."`)을 작성할 때는 항상 적절한 형식을 갖춘 코드를 사용하십시오. 세미콜론을 사용하여 구문을 분리하고 가독성을 유지하십시오. 약 3줄 이상의 구문이 필요한 경우, 대신 'heredoc' 문법을 사용하십시오:

```bash
python << 'EOF'
from dotenv import load_dotenv
load_dotenv(".env")

import videodb
conn = videodb.connect()
coll = conn.get_collection()
print(f"Videos: {len(coll.get_videos())}")
EOF
```

### 설정 (Setup)

사용자가 "videodb 설정(setup videodb)" 또는 이와 유사한 요청을 할 경우:

### 1. SDK 설치

```bash
pip install "videodb[capture]" python-dotenv
```

Linux에서 `videodb[capture]` 설치에 실패하면, capture 옵션 없이 설치하십시오:

```bash
pip install videodb python-dotenv
```

### 2. API 키 구성

사용자는 다음 중 **하나의** 방법을 사용하여 `VIDEO_DB_API_KEY`를 설정해야 합니다:

- **터미널에서 내보내기** (Claude 시작 전): `export VIDEO_DB_API_KEY=your-key`
- **프로젝트 `.env` 파일**: 프로젝트의 `.env` 파일에 `VIDEO_DB_API_KEY=your-key` 저장

https://console.videodb.io 에서 무료 API 키를 받을 수 있습니다 (카드 등록 없이 50회 업로드 무료).

**주의**: API 키를 직접 읽거나, 쓰거나, 다루지 마십시오. 항상 사용자가 직접 설정하게 하십시오.

### 퀵 레퍼런스 (Quick Reference)

### 미디어 업로드

```python
# URL
video = coll.upload(url="https://example.com/video.mp4")

# YouTube
video = coll.upload(url="https://www.youtube.com/watch?v=VIDEO_ID")

# 로컬 파일
video = coll.upload(file_path="/path/to/video.mp4")
```

### 스크립트(Transcript) + 자막

```python
# force=True를 사용하면 비디오가 이미 인덱싱되어 있더라도 에러를 무시합니다.
video.index_spoken_words(force=True)
text = video.get_transcript_text()
stream_url = video.add_subtitle()
```

### 비디오 내부 검색

```python
from videodb.exceptions import InvalidRequestError

video.index_spoken_words(force=True)

# 검색 결과가 없는 경우 search()는 InvalidRequestError를 발생시킵니다.
# 항상 try/except 구문으로 감싸고, "No results found"는 결과 없음으로 처리하십시오.
try:
    results = video.search("product demo")
    shots = results.get_shots()
    stream_url = results.compile()
except InvalidRequestError as e:
    if "No results found" in str(e):
        shots = []
    else:
        raise
```

### 장면(Scene) 검색

```python
import re
from videodb import SearchType, IndexType, SceneExtractionType
from videodb.exceptions import InvalidRequestError

# index_scenes()에는 force 파라미터가 없습니다. 즉, 장면 인덱스가 이미 존재하면 에러를 발생시킵니다.
# 에러 메시지에서 기존 인덱스 ID를 추출하십시오.
try:
    scene_index_id = video.index_scenes(
        extraction_type=SceneExtractionType.shot_based,
        prompt="이 장면의 시각적 콘텐츠를 설명하십시오.",
    )
except Exception as e:
    match = re.search(r"id\s+([a-f0-9]+)", str(e))
    if match:
        scene_index_id = match.group(1)
    else:
        raise

# 관련성이 낮은 노이즈를 필터링하기 위해 score_threshold를 사용하십시오 (권장: 0.3 이상)
try:
    results = video.search(
        query="칠판에 글씨를 쓰는 사람",
        search_type=SearchType.semantic,
        index_type=IndexType.scene,
        scene_index_id=scene_index_id,
        score_threshold=0.3,
    )
    shots = results.get_shots()
    stream_url = results.compile()
except InvalidRequestError as e:
    if "No results found" in str(e):
        shots = []
    else:
        raise
```

### 타임라인 편집

**중요:** 타임라인을 구축하기 전에 항상 타임스탬프를 검증하십시오:
- `start`는 반드시 0 이상이어야 함 (음수 값은 오류 없이 받아들여지나 깨진 결과를 생성함)
- `start`는 반드시 `end`보다 작아야 함
- `end`는 반드시 `video.length`보다 작거나 같아야 함

```python
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, TextAsset, TextStyle

timeline = Timeline(conn)
timeline.add_inline(VideoAsset(asset_id=video.id, start=10, end=30))
timeline.add_overlay(0, TextAsset(text="The End", duration=3, style=TextStyle(fontsize=36)))
stream_url = timeline.generate_stream()
```

### 비디오 트랜스코딩 (해상도 / 품질 변경)

```python
from videodb import TranscodeMode, VideoConfig, AudioConfig

# 서버 측에서 해상도, 품질 또는 가로세로 비율 변환
job_id = conn.transcode(
    source="https://example.com/video.mp4",
    callback_url="https://example.com/webhook",
    mode=TranscodeMode.economy,
    video_config=VideoConfig(resolution=720, quality=23, aspect_ratio="16:9"),
    audio_config=AudioConfig(mute=False),
)
```

### 가로세로 비율 재설정(Reframe) (소셜 플랫폼용)

**경고:** `reframe()`은 속도가 느린 서버 측 작업입니다. 긴 비디오의 경우 몇 분이 소요될 수 있으며 타임아웃이 발생할 수 있습니다. 베스트 프랙티스:
- 가급적 `start`/`end`를 사용하여 짧은 구간으로 제한하십시오.
- 전체 길이에 대해서는 비동기 처리를 위해 `callback_url`을 사용하십시오.
- 먼저 `Timeline`에서 비디오를 자른(Trim) 다음, 짧아진 결과물에 대해 reframe을 수행하십시오.

```python
from videodb import ReframeMode

# 가급적 짧은 세그먼트의 비율을 재설정하는 것을 선호하십시오:
reframed = video.reframe(start=0, end=60, target="vertical", mode=ReframeMode.smart)

# 전체 비디오에 대한 비동기 재설정 (None 반환, 결과는 웹훅을 통해 수신):
video.reframe(target="vertical", callback_url="https://example.com/webhook")

# 프리셋: "vertical" (9:16), "square" (1:1), "landscape" (16:9)
reframed = video.reframe(start=0, end=60, target="square")

# 커스텀 크기 설정
reframed = video.reframe(start=0, end=60, target={"width": 1280, "height": 720})
```

### 생성 미디어 (Generative media)

```python
image = coll.generate_image(
    prompt="산 너머로 지는 석양",
    aspect_ratio="16:9",
)
```

## 에러 처리

```python
from videodb.exceptions import AuthenticationError, InvalidRequestError

try:
    conn = videodb.connect()
except AuthenticationError:
    print("VIDEO_DB_API_KEY를 확인하십시오")

try:
    video = coll.upload(url="https://example.com/video.mp4")
except InvalidRequestError as e:
    print(f"업로드 실패: {e}")
```

### 흔히 발생하는 문제점

| 상황 | 에러 메시지 | 해결 방법 |
|----------|--------------|----------|
| 이미 인덱싱된 비디오를 인덱싱하려고 함 | `Spoken word index for video already exists` | 이미 인덱싱된 경우 건너뛰기 위해 `video.index_spoken_words(force=True)` 사용 |
| 장면 인덱스가 이미 존재함 | `Scene index with id XXXX already exists` | `re.search(r"id\s+([a-f0-9]+)", str(e))`를 사용하여 에러 메시지에서 기존 `scene_index_id` 추출 |
| 검색 결과가 없음 | `InvalidRequestError: No results found` | 예외를 포착하여 빈 결과(`shots = []`)로 처리 |
| Reframe 작업 타임아웃 | 긴 비디오 작업 시 무기한 대기 발생 | `start`/`end`로 세그먼트를 제한하거나, 비동기 처리를 위해 `callback_url` 전달 |
| 타임라인에서 음수 타임스탬프 사용 | 오류 없이 깨진 스트림 생성 | `VideoAsset`을 생성하기 전 항상 `start >= 0`인지 검증 |
| `generate_video()` / `create_collection()` 실패 | `Operation not allowed` 또는 `maximum limit` | 요금제에 따른 제한 기능임 — 사용자에게 요금제 한도에 대해 안내 |

## 예시

### 전형적인 프롬프트
- "데스크톱 캡처를 시작하고 비밀번호 필드가 나타나면 알림을 보내줘."
- "내 세션을 녹화하고 종료 시 실행 가능한 요약을 생성해줘."
- "이 파일을 수집하고 재생 가능한 스트림 링크를 반환해줘."
- "이 폴더를 인덱싱하고 사람이 나오는 모든 장면을 찾아서 타임스탬프를 알려줘."
- "자막을 생성하고, 인코딩해서 넣은 다음 잔잔한 배경 음악을 추가해줘."
- "이 RTSP URL을 연결하고 해당 구역에 사람이 들어오면 알림을 보내줘."

### 화면 녹화 (데스크톱 캡처)

녹화 세션 동안 WebSocket 이벤트를 캡처하려면 `ws_listener.py`를 사용하십시오. 데스크톱 캡처는 **macOS**만 지원합니다.

#### 빠른 시작

1. **상태 디렉토리 선택**: `STATE_DIR="${VIDEODB_EVENTS_DIR:-$HOME/.local/state/videodb}"`
2. **리스너 시작**: `VIDEODB_EVENTS_DIR="$STATE_DIR" python scripts/ws_listener.py --clear "$STATE_DIR" &`
3. **WebSocket ID 가져오기**: `cat "$STATE_DIR/videodb_ws_id"`
4. **캡처 코드 실행** (전체 워크플로우는 reference/capture.md 참조)
5. **이벤트 기록 위치**: `$STATE_DIR/videodb_events.jsonl`

새로운 캡처 작업을 시작할 때마다 `--clear` 옵션을 사용하여 이전 세션의 스크립트나 시각적 이벤트가 새 세션에 섞이지 않도록 하십시오.

#### 이벤트 쿼리

```python
import json
import os
import time
from pathlib import Path

events_dir = Path(os.environ.get("VIDEODB_EVENTS_DIR", Path.home() / ".local" / "state" / "videodb"))
events_file = events_dir / "videodb_events.jsonl"
events = []

if events_file.exists():
    with events_file.open(encoding="utf-8") as handle:
        for line in handle:
            try:
                events.append(json.loads(line))
            except json.JSONDecodeError:
                continue

transcripts = [e["data"]["text"] for e in events if e.get("channel") == "transcript"]
cutoff = time.time() - 300
recent_visual = [
    e for e in events
    if e.get("channel") == "visual_index" and e["unix_ts"] > cutoff
]
```

## 추가 문서

참조 문서는 이 SKILL.md 파일과 같은 위치의 `reference/` 디렉토리에 있습니다. 필요한 경우 Glob 도구를 사용하여 찾으십시오.

- [reference/api-reference.md](reference/api-reference.md) - VideoDB Python SDK API 전체 참조
- [reference/search.md](reference/search.md) - 비디오 검색(음성 및 장면 기반) 상세 가이드
- [reference/editor.md](reference/editor.md) - 타임라인 편집, 에셋 및 구성
- [reference/streaming.md](reference/streaming.md) - HLS 스트리밍 및 즉시 재생
- [reference/generative.md](reference/generative.md) - AI 기반 미디어 생성 (이미지, 비디오, 오디오)
- [reference/rtstream.md](reference/rtstream.md) - 라이브 스트림 수집 워크플로우 (RTSP/RTMP)
- [reference/rtstream-reference.md](reference/rtstream-reference.md) - RTStream SDK 메서드 및 AI 파이프라인
- [reference/capture.md](reference/capture.md) - 데스크톱 캡처 워크플로우
- [reference/capture-reference.md](reference/capture-reference.md) - 캡처 SDK 및 WebSocket 이벤트
- [reference/use-cases.md](reference/use-cases.md) - 일반적인 비디오 처리 패턴 및 예시


**ffmpeg, moviepy 또는 로컬 인코딩 도구를 사용하지 마십시오.** VideoDB가 다음 기능들을 서버 측에서 지원합니다 — 자르기(Trimming), 클립 결합, 오디오 또는 음악 오버레이, 자막 추가, 텍스트/이미지 오버레이, 트랜스코딩, 해상도 변경, 가로세로 비율 변환, 플랫폼 요구사항에 따른 크기 조정, 스크립트 추출 및 미디어 생성. reference/editor.md의 Limitations 섹션에 상기된 로컬 전용 인코딩(장면 전환, 속도 조절, 크롭/줌, 컬러 그레이딩, 볼륨 리믹싱 등)의 경우에만 로컬 도구를 사용하십시오.

### 상황별 도구 선택

| 문제 상황 | VideoDB 해결 방법 |
|---------|-----------------|
| 플랫폼이 비디오의 가로세로 비율이나 해상도를 거부함 | `video.reframe()` 또는 `VideoConfig`를 포함한 `conn.transcode()` |
| Twitter/Instagram/TikTok용으로 크기를 조정해야 함 | `video.reframe(target="vertical")` 또는 `target="square"` |
| 해상도를 변경해야 함 (예: 1080p → 720p) | `VideoConfig(resolution=720)`를 포함한 `conn.transcode()` |
| 비디오에 오디오/음악을 겹쳐야 함 | `Timeline` 상의 `AudioAsset` |
| 자막을 추가해야 함 | `video.add_subtitle()` 또는 `CaptionAsset` |
| 클립을 결합하거나 잘라야 함 | `Timeline` 상의 `VideoAsset` |
| 나레이션, 음악 또는 효과음을 생성해야 함 | `coll.generate_voice()`, `generate_music()`, `generate_sound_effect()` |

## 출처 (Provenance)

이 스킬의 참조 자료는 로컬 디렉토리 `skills/videodb/reference/`에 포함되어 있습니다. 런타임 중에 외부 저장소 링크를 따르는 대신 위의 로컬 복사본을 사용하십시오.

**유지관리:** [VideoDB](https://www.videodb.io/)
