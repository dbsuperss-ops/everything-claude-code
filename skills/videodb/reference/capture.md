# 캡처 가이드 (Capture Guide)

## 개요

VideoDB Capture는 AI 처리를 통한 실시간 화면 및 오디오 녹화 기능을 제공합니다. 현재 데스크톱 캡처는 **macOS**만 지원합니다.

코드 수준의 상세 내용(SDK 메서드, 이벤트 구조, AI 파이프라인 등)은 [capture-reference.md](capture-reference.md)를 참조하십시오.

## 빠른 시작

1. **WebSocket 리스너 시작**: `python scripts/ws_listener.py --clear &`
2. **캡처 코드 실행** (아래 '전체 캡처 워크플로우' 참조)
3. **이벤트 기록 위치**: `/tmp/videodb_events.jsonl`

---

## 전체 캡처 워크플로우

웹훅이나 폴링이 필요하지 않습니다. 세션 수명 주기를 포함한 모든 이벤트는 WebSocket을 통해 전달됩니다.

> **중요:** 캡처가 진행되는 동안 `CaptureClient`는 반드시 실행 상태를 유지해야 합니다. 셰션은 화면/오디오 데이터를 VideoDB로 스트리밍하는 로컬 녹화용 바이너리를 실행합니다. `CaptureClient`를 생성한 Python 프로세스가 종료되면 녹화용 바이너리도 강제 종료되어 캡처가 소리 없이 중단됩니다. 캡처 코드는 반드시 **장기 실행 백그라운드 프로세스** (예: `nohup python capture_script.py &`)로 실행하고, 명시적으로 중단할 때까지 `asyncio.Event`와 신호 처리(`SIGINT`/`SIGTERM`)를 사용하여 프로세스를 유지하십시오.

1. **WebSocket 리스너를 백그라운드에서 시작**합니다. 이때 `--clear` 플래그를 사용하여 이전 이벤트를 삭제하십시오. 리스너가 WebSocket ID 파일을 생성할 때까지 잠시 기다립니다.

2. **WebSocket ID를 읽습니다**. 이 ID는 캡처 세션 및 AI 파이프라인 설정에 필요합니다.

3. **캡처 세션을 생성**하고 데스크톱 클라이언트를 위한 클라이언트 토큰을 생성합니다.

4. 생성된 토큰으로 **CaptureClient를 초기화**합니다. 마이크 및 화면 캡처 권한을 요청합니다.

5. **채널 목록을 확인하고 선택**합니다 (마이크, 디스플레이, 시스템 오디오 등). 비디오로 저장하고 싶은 채널에는 `store = True`를 설정하십시오.

6. 선택한 채널로 **세션을 시작**합니다.

7. `capture_session.active` 이벤트가 나타날 때까지 이벤트를 읽으며 **세션 활성화를 대기**합니다. 이 이벤트에는 `rtstreams` 배열이 포함되어 있습니다. 다른 스크립트에서 읽을 수 있도록 세션 정보(세션 ID, RTStream ID 등)를 파일(예: `/tmp/videodb_capture_info.json`)에 저장하십시오.

8. **프로세스를 유지합니다**. 명시적으로 중지될 때까지 프로세스를 차단하기 위해 `SIGINT`/`SIGTERM` 신호 처리기와 함께 `asyncio.Event`를 사용하십시오. 나중에 `kill $(cat /tmp/videodb_capture_pid)` 명령으로 프로세스를 중단할 수 있도록 PID 파일(예: `/tmp/videodb_capture_pid`)을 작성하십시오. PID 파일은 실행할 때마다 덮어써서 항상 올바른 PID가 유지되도록 해야 합니다.

9. 오디오 및 비주얼 인덱싱을 위해 각 RTStream에 대한 **AI 파이프라인을 시작**합니다 (별도의 명령이나 스크립트 권장). 저장된 세션 정보 파일에서 RTStream ID를 읽어오십시오.

10. 사용 사례에 따라 실시간 이벤트를 읽는 **커스텀 이벤트 처리 로직을 작성**합니다 (별도의 명령이나 스크립트 권장). 예시:
    - `visual_index`에서 "Slack"이 언급되면 Slack 활동 기록
    - `audio_index` 이벤트가 발생하면 대화 내용 요약
    - `transcript`에 특정 키워드가 나타나면 알림 트리거
    - 화면 설명을 통해 애플리케이션 사용 현황 추적

11. 작업이 완료되면 **캡처를 중단**합니다 — 캡처 프로세스에 SIGTERM 신호를 보냅니다. 프로세스의 신호 처리기에서 `client.stop_capture()`와 `client.shutdown()`이 호출되어야 합니다.

12. `capture_session.exported` 이벤트가 나타날 때까지 이벤트를 읽으며 **내보내기(Export) 완료를 대기**합니다. 이 이벤트에는 `exported_video_id`, `stream_url`, `player_url`이 포함됩니다. 캡처 중단 후 수 초 정도 소요될 수 있습니다.

13. 내보내기 이벤트를 수신한 후 **WebSocket 리스너를 중단**합니다. `kill $(cat /tmp/videodb_ws_pid)` 명령을 사용하여 안전하게 종료하십시오.

---

## 종료 순서

모든 이벤트가 정상적으로 캡처되도록 하기 위해 올바른 종료 순서를 지키는 것이 중요합니다:

1. **캡처 세션 중단** — `client.stop_capture()` 호출 후 `client.shutdown()` 호출
2. **내보내기 이벤트 대기** — `/tmp/videodb_events.jsonl` 파일에서 `capture_session.exported`가 나타날 때까지 대기
3. **WebSocket 리스너 중단** — `kill $(cat /tmp/videodb_ws_pid)` 호출

내보내기 이벤트를 받기 전에 WebSocket 리스너를 종료하지 마십시오. 종료할 경우 최종 비디오 URL 정보를 놓치게 됩니다.

---

## 스크립트

| 스크립트 | 설명 |
|--------|-------------|
| `scripts/ws_listener.py` | WebSocket 이벤트 리스너 (JSONL 형식으로 기록) |

### ws_listener.py 사용법

```bash
# 백그라운드에서 리스너 시작 (기존 이벤트 뒤에 추가)
python scripts/ws_listener.py &

# --clear 옵션과 함께 시작 (새 세션, 이전 이벤트 삭제)
python scripts/ws_listener.py --clear &

# 커스텀 출력 디렉토리 지정
python scripts/ws_listener.py --clear /path/to/events &

# 리스너 중단
kill $(cat /tmp/videodb_ws_pid)
```

**옵션:**
- `--clear`: 시작 전 이벤트 파일을 삭제합니다. 새로운 캡처 세션을 시작할 때 사용하십시오.

**출력 파일:**
- `videodb_events.jsonl` - 모든 WebSocket 이벤트
- `videodb_ws_id` - WebSocket 연결 ID (`ws_connection_id` 파라미터용)
- `videodb_ws_pid` - 프로세스 ID (리스너 중단용)

**기능:**
- 연결 끊김 시 지수 백오프(Exponential backoff)를 통한 자동 재연결
- SIGINT/SIGTERM 수신 시 안전한 종료
- 프로세스 관리를 위한 PID 파일 생성
- 연결 상태 로깅
