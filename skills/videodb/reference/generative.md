# 생성 미디어 가이드 (Generative Media Guide)

VideoDB는 이미지, 비디오, 음악, 효과음, 음성 및 텍스트 콘텐츠의 AI 기반 생성 기능을 제공합니다. 모든 생성 메서드는 **Collection** 객체에 정의되어 있습니다.

## 전제 조건

생성 메서드를 호출하기 전에 먼저 연결(Connection)을 설정하고 컬렉션 참조를 확보해야 합니다:

```python
import videodb

conn = videodb.connect()
coll = conn.get_collection()
```

## 이미지 생성 (Image Generation)

텍스트 프롬프트로부터 이미지를 생성합니다:

```python
image = coll.generate_image(
    prompt="날아다니는 자동차가 있는 석양 무렵의 미래 지향적인 도시 풍경",
    aspect_ratio="16:9",
)

# 생성된 이미지 확인
print(image.id)
print(image.generate_url())  # 서명된 다운로드 URL 반환
```

### generate_image 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `prompt` | `str` | 필수 | 생성할 이미지에 대한 텍스트 설명 |
| `aspect_ratio` | `str` | `"1:1"` | 가로세로 비율: `"1:1"`, `"9:16"`, `"16:9"`, `"4:3"`, 또는 `"3:4"` |
| `callback_url` | `str\|None` | `None` | 비동기 콜백을 받을 URL |

`.id`, `.name`, `.collection_id` 속성을 가진 `Image` 객체를 반환합니다. 생성된 이미지의 경우 `.url` 속성이 `None`일 수 있으므로, 항상 `image.generate_url()`을 사용하여 신뢰할 수 있는 서명된 다운로드 URL을 가져오십시오.

> **참고:** `Video` 객체(`.generate_stream()` 사용)와 달리, `Image` 객체는 `generate_url()`을 사용하여 이미지 URL을 검색합니다. `.url` 속성은 썸네일 등 일부 이미지 유형에 대해서만 채워집니다.

## 비디오 생성 (Video Generation)

텍스트 프롬프트로부터 짧은 비디오 클립을 생성합니다:

```python
video = coll.generate_video(
    prompt="정원에서 꽃이 피어나는 타임랩스 영상",
    duration=5,
)

stream_url = video.generate_stream()
video.play()
```

### generate_video 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `prompt` | `str` | 필수 | 생성할 비디오에 대한 텍스트 설명 |
| `duration` | `int` | `5` | 초 단위 재생 시간 (정수값이어야 하며, 5~8초 권장) |
| `callback_url` | `str\|None` | `None` | 비동기 콜백을 받을 URL |

`Video` 객체를 반환합니다. 생성된 비디오는 자동으로 컬렉션에 추가되며, 업로드된 다른 비디오와 마찬가지로 타임라인, 검색 및 컴파일 작업에 사용할 수 있습니다.

## 오디오 생성 (Audio Generation)

VideoDB는 오디오 유형에 따라 세 가지 독립적인 메서드를 제공합니다.

### 음악 (Music)

텍스트 설명으로부터 배경 음악을 생성합니다:

```python
music = coll.generate_music(
    prompt="테크 데모에 어울리는 비트감이 있는 경쾌한 일렉트로닉 음악",
    duration=30,
)

print(music.id)
```

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `prompt` | `str` | 필수 | 음악에 대한 텍스트 설명 |
| `duration` | `int` | `5` | 초 단위 재생 시간 |
| `callback_url` | `str\|None` | `None` | 비동기 콜백을 받을 URL |

### 효과음 (Sound Effects)

특정 효과음을 생성합니다:

```python
sfx = coll.generate_sound_effect(
    prompt="폭우와 멀리서 들리는 천둥소리가 포함된 뇌우 소리",
    duration=10,
)
```

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `prompt` | `str` | 필수 | 효과음에 대한 텍스트 설명 |
| `duration` | `int` | `2` | 초 단위 재생 시간 |
| `config` | `dict` | `{}` | 추가 구성 설정 |
| `callback_url` | `str\|None` | `None` | 비동기 콜백을 받을 URL |

### 음성 (텍스트-음성 변환, TTS)

텍스트로부터 음성을 생성합니다:

```python
voice = coll.generate_voice(
    text="제품 데모에 오신 것을 환영합니다. 오늘은 주요 기능들을 살펴보겠습니다.",
    voice_name="Default",
)
```

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `text` | `str` | 필수 | 음성으로 변환할 텍스트 |
| `voice_name` | `str` | `"Default"` | 사용할 목소리 이름 |
| `config` | `dict` | `{}` | 추가 구성 설정 |
| `callback_url` | `str\|None` | `None` | 비동기 콜백을 받을 URL |

세 가지 오디오 메서드 모두 `.id`, `.name`, `.length`, `.collection_id` 속성을 가진 `Audio` 객체를 반환합니다.

## 텍스트 생성 (LLM 연동)

`coll.generate_text()`를 사용하여 LLM 분석을 수행하십시오. 이는 **Collection 수준**의 메서드입니다 -- 프롬프트 문자열에 분석 대상 컨텍스트(스크립트, 설명 등)를 직접 전달하십시오.

```python
# 먼저 비디오에서 스크립트 텍스트 가져오기
transcript_text = video.get_transcript_text()

# 컬렉션 LLM을 사용하여 분석 생성
result = coll.generate_text(
    prompt=f"이 비디오에서 논의된 주요 내용을 요약하십시오:\n{transcript_text}",
    model_name="pro",
)

print(result["output"])
```

### generate_text 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `prompt` | `str` | 필수 | LLM을 위한 컨텍스트가 포함된 프롬프트 |
| `model_name` | `str` | `"basic"` | 모델 등급: `"basic"`, `"pro"`, 또는 `"ultra"` |
| `response_type` | `str` | `"text"` | 응답 형식: `"text"` 또는 `"json"` |

`output` 키를 가진 `dict` 객체를 반환합니다. `response_type="text"`인 경우 `output`은 `str`이며, `response_type="json"`인 경우 `output`은 `dict`입니다.

```python
result = coll.generate_text(prompt="이 내용을 요약하십시오", model_name="pro")
print(result["output"])  # 실제 텍스트 또는 딕셔너리 데이터에 접근
```

### LLM으로 장면 분석하기

장면 추출과 텍스트 생성을 조합할 수 있습니다:

```python
from videodb import SceneExtractionType

# 1. 먼저 장면 인덱싱 수행
scenes = video.index_scenes(
    extraction_type=SceneExtractionType.time_based,
    extraction_config={"time": 10},
    prompt="이 장면의 시각적 콘텐츠를 설명하십시오.",
)

# 2. 음성 컨텍스트를 위한 스크립트 가져오기
transcript_text = video.get_transcript_text()
scene_descriptions = []
for scene in scenes:
    if isinstance(scene, dict):
        description = scene.get("description") or scene.get("summary")
    else:
        description = getattr(scene, "description", None) or getattr(scene, "summary", None)
    scene_descriptions.append(description or str(scene))

scenes_text = "\n".join(scene_descriptions)

# 3. 컬렉션 LLM으로 분석
result = coll.generate_text(
    prompt=(
        f"비디오 스크립트 내용:\n{transcript_text}\n\n"
        f"시각적 장면 설명:\n{scenes_text}\n\n"
        "위의 음성 및 시각적 내용을 바탕으로 다루어진 주요 주제들을 설명하십시오."
    ),
    model_name="pro",
)
print(result["output"])
```

## 더빙 및 번역 (Dubbing and Translation)

### 비디오 더빙

컬렉션 메서드를 사용하여 비디오를 다른 언어로 더빙합니다:

```python
dubbed_video = coll.dub_video(
    video_id=video.id,
    language_code="es",  # 스페인어
)

dubbed_video.play()
```

### dub_video 파라미터

| 파라미터 | 타입 | 기본값 | 설명 |
|-----------|------|---------|-------------|
| `video_id` | `str` | 필수 | 더빙할 비디오의 ID |
| `language_code` | `str` | 필수 | 대상 언어 코드 (예: `"es"`, `"fr"`, `"de"`, `"ko"` 등) |
| `callback_url` | `str\|None` | `None` | 비동기 콜백을 받을 URL |

더빙된 콘텐츠를 포함한 `Video` 객체를 반환합니다.

### 스크립트 번역

더빙 없이 비디오의 스크립트만 번역합니다:

```python
translated = video.translate_transcript(
    language="Spanish",
    additional_notes="격식 있는 어조를 사용하십시오",
)

for entry in translated:
    print(entry)
```

**지원 언어**에는 다음이 포함됩니다: `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `hi`, `ar` 등.

## 전체 워크플로우 예시

### 비디오 나레이션 생성하기

```python
import videodb

conn = videodb.connect()
coll = conn.get_collection()
video = coll.get_video("your-video-id")

# 1. 스크립트 가져오기
transcript_text = video.get_transcript_text()

# 2. 컬렉션 LLM을 사용하여 나레이션 스크립트 작성
result = coll.generate_text(
    prompt=(
        f"다음 비디오 콘텐츠를 위한 전문적인 나레이션 스크립트를 작성하십시오:\n"
        f"{transcript_text[:2000]}"
    ),
    model_name="pro",
)
script = result["output"]

# 3. 스크립트를 음성으로 변환
narration = coll.generate_voice(text=script)
print(f"나레이션 오디오 ID: {narration.id}")
```

### 프롬프트로부터 썸네일 생성하기

```python
thumbnail = coll.generate_image(
    prompt="데이터 분석 대시보드를 보여주는 전문적인 비디오 썸네일, 현대적인 디자인",
    aspect_ratio="16:9",
)
print(f"썸네일 URL: {thumbnail.generate_url()}")
```

### 비디오에 생성된 음악 추가하기

```python
import videodb
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, AudioAsset

conn = videodb.connect()
coll = conn.get_collection()
video = coll.get_video("your-video-id")

# 1. 배경 음악 생성
music = coll.generate_music(
    prompt="튜토리얼 비디오를 위한 침착하고 잔잔한 배경 음악",
    duration=60,
)

# 2. 비디오 + 음악 오버레이를 포함한 타임라인 구축
timeline = Timeline(conn)
timeline.add_inline(VideoAsset(asset_id=video.id))
timeline.add_overlay(0, AudioAsset(asset_id=music.id, disable_other_tracks=False))

# 3. 스트림 생성
stream_url = timeline.generate_stream()
print(f"음악이 추가된 비디오 URL: {stream_url}")
```

### 구조화된 JSON 출력 받기

```python
transcript_text = video.get_transcript_text()

result = coll.generate_text(
    prompt=(
        f"스크립트 내용:\n{transcript_text}\n\n"
        "다음 키를 포함한 JSON 객체를 반환하십시오: summary(요약), topics(주제 배열), action_items(실행 항목 배열)."
    ),
    model_name="pro",
    response_type="json",
)

# response_type="json"인 경우 result["output"]은 딕셔너리입니다.
print(result["output"]["summary"])
print(result["output"]["topics"])
```

## 팁

- **생성된 미디어는 영구적으로 저장됨**: 모든 생성된 콘텐츠는 컬렉션에 저장되어 재사용할 수 있습니다.
- **세 가지 오디오 메서드 구분**: 배경 음악은 `generate_music()`, 효과음은 `generate_sound_effect()`, 텍스트-음성 변환은 `generate_voice()`를 사용하십시오. 통합된 `generate_audio()` 메서드는 없습니다.
- **텍스트 생성은 컬렉션 수준 작업임**: `coll.generate_text()`는 자동으로 비디오 내용에 접근하지 못합니다. 먼저 `video.get_transcript_text()`로 스크립트를 가져와 프롬프트에 포함시켜야 합니다.
- **모델 등급**: `"basic"`은 가장 빠르고, `"pro"`는 균형 잡힌 속도와 품질을 제공하며, `"ultra"`는 가장 높은 품질을 제공합니다. 대부분의 분석 작업에는 `"pro"`를 권장합니다.
- **여러 생성 유형 조합**: 오버레이용 이미지, 배경 음악, 나레이션용 음성을 각각 생성한 다음 타임라인을 사용하여 조합하십시오 ([editor.md](editor.md) 참조).
- **프롬프트 품질의 중요성**: 모든 생성 유형에서 더 구체적이고 묘사가 풍부한 프롬프트가 더 좋은 결과를 만듭니다.
- **이미지 가로세로 비율**: `"1:1"`, `"9:16"`, `"16:9"`, `"4:3"`, 또는 `"3:4"` 중에서 선택할 수 있습니다.
