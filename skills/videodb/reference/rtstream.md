# RTStream 가이드 (RTStream Guide)

## 개요

RTStream은 라이브 비디오 스트림(RTSP/RTMP) 및 데스크톱 캡처 세션의 실시간 수집 기능을 제공합니다. 연결이 완료되면 라이브 소스로부터 콘텐츠를 녹화, 인덱싱, 검색 및 내보내기(Export)할 수 있습니다.

코드 수준의 상세 내용(SDK 메서드, 파라미터, 예시 등)은 [rtstream-reference.md](rtstream-reference.md)를 참조하십시오.

## 사용 사례

- **보안 및 모니터링**: RTSP 카메라를 연결하고, 이벤트를 감지하며, 알림을 트리거합니다.
- **라이브 방송**: RTMP 스트림을 수집하고, 실시간으로 인덱싱하며, 즉각적인 검색 기능을 제공합니다.
- **회의 녹화**: 데스크톱 화면과 오디오를 캡처하고, 실시간으로 스크립트를 추출하며, 녹화본을 내보냅니다.
- **이벤트 처리**: 라이브 피드를 모니터링하고, AI 분석을 실행하며, 감지된 콘텐츠에 대응합니다.

## 빠른 시작

1. **라이브 스트림에 연결**(RTSP/RTMP URL 사용)하거나 캡처 세션으로부터 RTStream을 가져옵니다.

2. **수집(Ingestion)을 시작**하여 라이브 콘텐츠 녹화를 시작합니다.

3. 실시간 인덱싱(오디오, 비주얼, 스크립트 추출)을 위해 **AI 파이프라인을 시작**합니다.

4. 라이브 AI 결과 및 알림 확인을 위해 WebSocket을 통해 **이벤트를 모니터링**합니다.

5. 작업이 끝나면 **수집을 중단**합니다.

6. 영구 저장 및 추가 처리를 위해 **비디오로 내보내기**를 수행합니다.

7. 특정 순간을 찾기 위해 **녹화본 검색**을 수행합니다.

## RTStream 소스

### RTSP/RTMP 스트림으로부터

라이브 비디오 소스에 직접 연결합니다:

```python
rtstream = coll.connect_rtstream(
    url="rtmp://your-stream-server/live/stream-key",
    name="내 라이브 스트림",
)
```

### 캡처 세션으로부터

데스크톱 캡처(마이크, 화면, 시스템 오디오)에서 RTStream을 가져옵니다:

```python
session = conn.get_capture_session(session_id)

mics = session.get_rtstream("mic")
displays = session.get_rtstream("screen")
system_audios = session.get_rtstream("system_audio")
```

캡처 세션 워크플로우에 대해서는 [capture.md](capture.md)를 참조하십시오.

---

## 스크립트

| 스크립트 | 설명 |
|--------|-------------|
| `scripts/ws_listener.py` | 실시간 AI 결과 확인을 위한 WebSocket 이벤트 리스너 |
