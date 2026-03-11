# RTStream 레퍼런스 (RTStream Reference)

RTStream 작업에 대한 코드 수준의 상세 내용입니다. 워크플로우 가이드는 [rtstream.md](rtstream.md)를 참조하십시오.
사용법 안내 및 워크플로우 선택에 대해서는 [../SKILL.md](../SKILL.md)를 먼저 확인하십시오.

이 문서는 [docs.videodb.io](https://docs.videodb.io/pages/ingest/live-streams/realtime-apis.md)의 내용을 기반으로 합니다.

---

## 컬렉션 RTStream 메서드

RTStream 관리를 위한 `Collection` 객체의 메서드들입니다:

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `coll.connect_rtstream(url, name, ...)` | `RTStream` | RTSP/RTMP URL로부터 새로운 RTStream 생성 |
| `coll.get_rtstream(id)` | `RTStream` | ID로 기존 RTStream 가져오기 |
| `coll.list_rtstreams(limit, offset, status, name, ordering)` | `List[RTStream]` | 컬렉션 내의 모든 RTStream 목록 표시 |
| `coll.search(query, namespace="rtstream")` | `RTStreamSearchResult` | 모든 RTStream에 대해 검색 수행 |

### RTStream 연결 (Connect RTStream)

```python
import videodb

conn = videodb.connect()
coll = conn.get_collection()

rtstream = coll.connect_rtstream(
    url="rtmp://your-stream-server/live/stream-key",
    name="내 라이브 스트림",
    media_types=["video"],  # 또는 ["audio", "video"]
    sample_rate=30,         # 선택 사항
    store=True,             # 내보내기를 위한 녹화 저장 활성화
    enable_transcript=True, # 선택 사항
    ws_connection_id=ws_id, # 선택 사항, 실시간 이벤트용
)
```

### 기존 RTStream 가져오기

```python
rtstream = coll.get_rtstream("rts-xxx")
```

### RTStream 목록 표시

```python
rtstreams = coll.list_rtstreams(
    limit=10,
    offset=0,
    status="connected",  # 선택적 필터
    name="meeting",      # 선택적 필터
    ordering="-created_at",
)

for rts in rtstreams:
    print(f"{rts.id}: {rts.name} - {rts.status}")
```

### 캡처 세션으로부터 가져오기

캡처 세션이 활성화된 후 RTStream 객체를 검색합니다:

```python
session = conn.get_capture_session(session_id)

mics = session.get_rtstream("mic")
displays = session.get_rtstream("screen")
system_audios = session.get_rtstream("system_audio")
```

또는 WebSocket의 `capture_session.active` 이벤트에서 `rtstreams` 데이터를 사용합니다:

```python
for rts in rtstreams:
    rtstream = coll.get_rtstream(rts["rtstream_id"])
```

---

## RTStream 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `rtstream.start()` | `None` | 데이터 수집 시작 |
| `rtstream.stop()` | `None` | 데이터 수집 중단 |
| `rtstream.generate_stream(start, end)` | `str` | 녹화된 구간의 스트리밍 실행 (Unix 타임스탬프 사용) |
| `rtstream.export(name=None)` | `RTStreamExportResult` | 영구 비디오로 내보내기 |
| `rtstream.index_visuals(prompt, ...)` | `RTStreamSceneIndex` | IT 분석을 통한 시각적 인덱스 생성 |
| `rtstream.index_audio(prompt, ...)` | `RTStreamSceneIndex` | LLM 요약을 통한 오디오 인덱스 생성 |
| `rtstream.list_scene_indexes()` | `List[RTStreamSceneIndex]` | 스트림의 모든 장면 인덱스 목록 표시 |
| `rtstream.get_scene_index(index_id)` | `RTStreamSceneIndex` | 특정 장면 인덱스 가져오기 |
| `rtstream.search(query, ...)` | `RTStreamSearchResult` | 인덱싱된 콘텐츠 검색 |
| `rtstream.start_transcript(ws_connection_id, engine)` | `dict` | 실시간 스크립트 추출(Transcription) 시작 |
| `rtstream.get_transcript(page, page_size, start, end, since)` | `dict` | 스크립트 페이지 가져오기 |
| `rtstream.stop_transcript(engine)` | `dict` | 스크립트 추출 중단 |

---

## 시작 및 중단

```python
# 데이터 수집 시작
rtstream.start()

# ... 스트림 녹화 중 ...

# 데이터 수집 중단
rtstream.stop()
```

---

## 스트림 URL 생성

녹화된 콘텐츠로부터 재생 스트림을 생성할 때는 Unix 타임스탬프(초 단위 오프셋 아님)를 사용합니다:

```python
import time

start_ts = time.time()
rtstream.start()

# 일정 시간 동안 녹화...
time.sleep(60)

end_ts = time.time()
rtstream.stop()

# 녹화된 세그먼트에 대한 스트림 URL 생성
stream_url = rtstream.generate_stream(start=start_ts, end=end_ts)
print(f"녹화된 스트림 URL: {stream_url}")
```

---

## 비디오로 내보내기 (Exporting)

녹화된 스트림을 컬렉션 내의 영구 비디오 파일로 내보냅니다:

```python
export_result = rtstream.export(name="회의 녹화 2024-01-15")

print(f"비디오 ID: {export_result.video_id}")
print(f"스트림 URL: {export_result.stream_url}")
print(f"플레이어 URL: {export_result.player_url}")
print(f"재생 시간: {export_result.duration}초")
```

### RTStreamExportResult 속성 (Properties)

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `video_id` | `str` | 내보내기 된 비디오의 ID |
| `stream_url` | `str` | HLS 스트림 URL |
| `player_url` | `str` | 웹 플레이어 URL |
| `name` | `str` | 비디오 이름 |
| `duration` | `float` | 초 단위 재생 시간 |

---

## AI 파이프라인 (AI Pipelines)

AI 파이프라인은 라이브 스트림을 처리하고 WebSocket을 통해 결과를 전송합니다.

### RTStream AI 파이프라인 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `rtstream.index_audio(prompt, batch_config, ...)` | `RTStreamSceneIndex` | LLM 요약을 사용한 오디오 인덱싱 시작 |
| `rtstream.index_visuals(prompt, batch_config, ...)` | `RTStreamSceneIndex` | 화면 콘텐츠의 시각적 인덱싱 시작 |

### 오디오 인덱싱

일정 간격으로 오디오 콘텐츠의 LLM 요약을 생성합니다:

```python
audio_index = rtstream.index_audio(
    prompt="논의되고 있는 내용을 요약하십시오",
    batch_config={"type": "word", "value": 50},
    model_name=None,       # 선택 사항
    name="meeting_audio",  # 선택 사항
    ws_connection_id=ws_id,
)
```

**오디오용 batch_config 옵션:**

| 타입(Type) | 값(Value) | 설명 |
|------|-------|-------------|
| `"word"` | 개수 | N 단어마다 세그먼트 생성 |
| `"sentence"` | 개수 | N 문장마다 세그먼트 생성 |
| `"time"` | 초 | N 초마다 세그먼트 생성 |

예시:
```python
{"type": "word", "value": 50}      # 50단어마다
{"type": "sentence", "value": 5}   # 5문장마다
{"type": "time", "value": 30}      # 30초마다
```

결과는 `audio_index` WebSocket 채널을 통해 들어옵니다.

### 시각적 인덱싱

시각적 콘텐츠의 AI 설명을 생성합니다:

```python
scene_index = rtstream.index_visuals(
    prompt="화면에서 일어나고 있는 일을 설명하십시오",
    batch_config={"type": "time", "value": 2, "frame_count": 5},
    model_name="basic",
    name="screen_monitor",  # 선택 사항
    ws_connection_id=ws_id,
)
```

**파라미터:**

| 파라미터 | 타입 | 설명 |
|-----------|------|-------------|
| `prompt` | `str` | AI 모델을 위한 명령 (구조화된 JSON 출력 지원) |
| `batch_config` | `dict` | 프레임 샘플링 제어 (아래 참조) |
| `model_name` | `str` | 모델 등급: `"mini"`, `"basic"`, `"pro"`, `"ultra"` |
| `name` | `str` | 인덱스 이름 (선택 사항) |
| `ws_connection_id` | `str` | 결과를 수신할 WebSocket 연결 ID |

**시각적 인덱싱용 batch_config:**

| 키(Key) | 타입 | 설명 |
|-----|------|-------------|
| `type` | `str` | 시각적 인덱싱에는 `"time"`만 지원됩니다. |
| `value` | `int` | 초 단위 윈도우 크기 |
| `frame_count` | `int` | 윈도우당 추출할 프레임 수 |

예시: `{"type": "time", "value": 2, "frame_count": 5}`는 2초마다 5개의 프레임을 샘플링하여 모델로 전송합니다.

**구조화된 JSON 출력:**

구조화된 응답을 받으려면 JSON 형식을 요청하는 프롬프트를 사용하십시오:

```python
scene_index = rtstream.index_visuals(
    prompt="""화면을 분석하고 다음을 포함하는 JSON 객체를 반환하십시오:
{
  "app_name": "활성화된 애플리케이션 이름",
  "activity": "사용자가 무엇을 하고 있는지",
  "ui_elements": ["공개된 UI 요소 목록"],
  "contains_text": true/false,
  "dominant_colors": ["주요 색상 목록"]
}
유효한 JSON만 반환하십시오.""",
    batch_config={"type": "time", "value": 3, "frame_count": 3},
    model_name="pro",
    ws_connection_id=ws_id,
)
```

결과는 `scene_index` WebSocket 채널을 통해 들어옵니다.

---

## Batch Config 요약

| 인덱싱 유형 | `type` 옵션 | `value` 의미 | 추가 키 |
|---------------|----------------|---------|------------|
| **오디오** | `"word"`, `"sentence"`, `"time"` | 단어/문장/초 수 | - |
| **비주얼** | `"time"` 전용 | 초 수 | `frame_count` |

예시:
```python
# 오디오: 50단어마다
{"type": "word", "value": 50}

# 오디오: 30초마다  
{"type": "time", "value": 30}

# 비주얼: 2초마다 5프레임
{"type": "time", "value": 2, "frame_count": 5}
```

---

## 스크립트 추출 (Transcription)

WebSocket을 통한 실시간 스크립트 추출:

```python
# 실시간 스크립트 추출 시작
rtstream.start_transcript(
    ws_connection_id=ws_id,
    engine=None,  # 선택 사항, 기본값은 "assemblyai"
)

# 스크립트 페이지 가져오기 (선택적 필터 사용 가능)
transcript = rtstream.get_transcript(
    page=1,
    page_size=100,
    start=None,   # 선택 사항: 시작 타임스탬프 필터
    end=None,     # 선택 사항: 종료 타임스탬프 필터
    since=None,   # 선택 사항: 폴링용, 이 타임스탬프 이후의 스크립트 가져오기
    engine=None,
)

# 스크립트 추출 중단
rtstream.stop_transcript(engine=None)
```

스크립트 결과는 `transcript` WebSocket 채널을 통해 들어옵니다.

---

## RTStreamSceneIndex

`index_audio()` 또는 `index_visuals()`를 호출하면 `RTStreamSceneIndex` 객체가 반환됩니다. 이 객체는 실행 중인 인덱스를 나타내며 장면 및 알림 관리를 위한 메서드를 제공합니다.

```python
# index_visuals는 RTStreamSceneIndex를 반환합니다.
scene_index = rtstream.index_visuals(
    prompt="화면에 무엇이 있는지 설명하십시오",
    ws_connection_id=ws_id,
)

# index_audio 또한 RTStreamSceneIndex를 반환합니다.
audio_index = rtstream.index_audio(
    prompt="논의 내용을 요약하십시오",
    ws_connection_id=ws_id,
)
```

### RTStreamSceneIndex 속성 (Properties)

| 속성 | 타입 | 설명 |
|----------|------|-------------|
| `rtstream_index_id` | `str` | 인덱스의 고유 ID |
| `rtstream_id` | `str` | 상위 RTStream의 ID |
| `extraction_type` | `str` | 추출 유형 (`time` 또는 `transcript`) |
| `extraction_config` | `dict` | 추출 구성 설정 |
| `prompt` | `str` | 분석에 사용된 프롬프트 |
| `name` | `str` | 인덱스의 이름 |
| `status` | `str` | 상태 (`connected`, `stopped`) |

### RTStreamSceneIndex 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `index.get_scenes(start, end, page, page_size)` | `dict` | 인덱싱된 장면 가져오기 |
| `index.start()` | `None` | 인덱스 시작/재개 |
| `index.stop()` | `None` | 인덱스 중단 |
| `index.create_alert(event_id, callback_url, ws_connection_id)` | `str` | 이벤트 감지를 위한 알림 생성 |
| `index.list_alerts()` | `list` | 이 인덱스의 모든 알림 목록 표시 |
| `index.enable_alert(alert_id)` | `None` | 알림 활성화 |
| `index.disable_alert(alert_id)` | `None` | 알림 비활성화 |

### 장면 가져오기

인덱스로부터 인덱싱된 장면들을 폴링합니다:

```python
result = scene_index.get_scenes(
    start=None,      # 선택 사항: 시작 타임스탬프
    end=None,        # 선택 사항: 종료 타임스탬프
    page=1,
    page_size=100,
)

for scene in result["scenes"]:
    print(f"[{scene['start']}-{scene['end']}] {scene['text']}")

if result["next_page"]:
    # 다음 페이지 가져오기 수행
    pass
```

### 장면 인덱스 관리

```python
# 스트림의 모든 인덱스 목록 표시
indexes = rtstream.list_scene_indexes()

# ID로 특정 인덱스 가져오기
scene_index = rtstream.get_scene_index(index_id)

# 인덱스 중단
scene_index.stop()

# 인덱스 재시작
scene_index.start()
```

---

## 이벤트 (Events)

이벤트는 재사용 가능한 감지 규칙입니다. 한 번 생성하면 알림을 통해 모든 인덱스에 연결할 수 있습니다.

### 연결(Connection) 이벤트 메서드

| 메서드 | 반환값 | 설명 |
|--------|---------|-------------|
| `conn.create_event(event_prompt, label)` | `str` (event_id) | 감지 이벤트 생성 |
| `conn.list_events()` | `list` | 모든 이벤트 목록 표시 |

### 이벤트 생성 예시

```python
event_id = conn.create_event(
    event_prompt="사용자가 Slack 애플리케이션을 열었습니다",
    label="slack_opened",
)
```

### 이벤트 목록 표시 예시

```python
events = conn.list_events()
for event in events:
    print(f"{event['event_id']}: {event['label']}")
```

---

## 알림 (Alerts)

알림은 실시간 통보를 위해 이벤트를 인덱스에 연결합니다. AI가 이벤트 설명과 일치하는 콘텐츠를 감지하면 알림이 전송됩니다.

### 알림 생성 예시

```python
# index_visuals로부터 RTStreamSceneIndex 가져오기
scene_index = rtstream.index_visuals(
    prompt="화면에 어떤 애플리케이션이 열려 있는지 설명하십시오",
    ws_connection_id=ws_id,
)

# 인덱스에 알림 생성
alert_id = scene_index.create_alert(
    event_id=event_id,
    callback_url="https://your-backend.com/alerts",  # 웹훅 전달용
    ws_connection_id=ws_id,  # WebSocket 전달용 (선택 사항)
)
```

**참고:** `callback_url`은 필수입니다. WebSocket 전달만 사용하는 경우 빈 문자열 `""`을 전달하십시오.

### 알림 관리 예시

```python
# 인덱스의 모든 알림 목록 표시
alerts = scene_index.list_alerts()

# 알림 활성화/비활성화
scene_index.disable_alert(alert_id)
scene_index.enable_alert(alert_id)
```

### 알림 전달 방식

| 방식 | 지연 시간 | 용도 |
|--------|---------|----------|
| WebSocket | 실시간 | 대시보드, 라이브 UI 등 |
| Webhook | 1초 미만 | 서버 간 통신, 자동화 등 |

### WebSocket 알림 이벤트 예시

```json
{
  "channel": "alert",
  "rtstream_id": "rts-xxx",
  "data": {
    "event_label": "slack_opened",
    "timestamp": 1710000012340,
    "text": "사용자가 Slack 애플리케이션을 열었습니다"
  }
}
```

### 웹훅(Webhook) 페이로드 예시

```json
{
  "event_id": "event-xxx",
  "label": "slack_opened",
  "confidence": 0.95,
  "explanation": "사용자가 Slack 애플리케이션을 열었습니다",
  "timestamp": "2024-01-15T10:30:45Z",
  "start_time": 1234.5,
  "end_time": 1238.0,
  "stream_url": "https://stream.videodb.io/v3/...",
  "player_url": "https://console.videodb.io/player?url=..."
}
```

---

## WebSocket 통합

모든 실시간 AI 결과는 WebSocket을 통해 전달됩니다. 다음 메서드들에 `ws_connection_id`를 전달하십시오:
- `rtstream.start_transcript()`
- `rtstream.index_audio()`
- `rtstream.index_visuals()`
- `scene_index.create_alert()`

### WebSocket 채널 목록

| 채널 | 소스 | 콘텐츠 |
|---------|--------|---------|
| `transcript` | `start_transcript()` | 실시간 음성-텍스트 변환 결과 |
| `scene_index` | `index_visuals()` | 시각적 분석 결과 |
| `audio_index` | `index_audio()` | 오디오 분석 결과 |
| `alert` | `create_alert()` | 알림 통보 |

WebSocket 이벤트 구조 및 ws_listener 사용법에 대해서는 [capture-reference.md](capture-reference.md)를 참조하십시오.

---

## 전체 워크플로우 예시

```python
import time
import videodb
from videodb.exceptions import InvalidRequestError

conn = videodb.connect()
coll = conn.get_collection()

# 1. 연결 및 녹화 시작
rtstream = coll.connect_rtstream(
    url="rtmp://your-stream-server/live/stream-key",
    name="주간 스탠드업 회의",
    store=True,
)
rtstream.start()

# 2. 회의 시간 동안 녹화 수행
start_ts = time.time()
time.sleep(1800)  # 30분 동안 진행
end_ts = time.time()
rtstream.stop()

# 캡처된 구간에 대해 즉시 재생 가능한 URL 생성
stream_url = rtstream.generate_stream(start=start_ts, end=end_ts)
print(f"녹화된 스트림 URL: {stream_url}")

# 3. 영구 비디오로 내보내기
export_result = rtstream.export(name="주간 스탠드업 녹화본")
print(f"내보내기 된 비디오 ID: {export_result.video_id}")

# 4. 검색을 위해 내보내기 된 비디오 음성 인덱싱
video = coll.get_video(export_result.video_id)
video.index_spoken_words(force=True)

# 5. 실행 항목(Action items) 검색
try:
    results = video.search("action items and next steps")
    stream_url = results.compile()
    print(f"실행 항목 클립 URL: {stream_url}")
except InvalidRequestError as exc:
    if "No results found" in str(exc):
        print("녹화본에서 실행 항목이 감지되지 않았습니다.")
    else:
        raise
```
