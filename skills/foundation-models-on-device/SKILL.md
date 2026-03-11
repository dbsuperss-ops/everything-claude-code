---
name: foundation-models-on-device
description: 온디바이스(On-device) LLM을 위한 Apple FoundationModels 프레임워크 — 텍스트 생성, @Generable을 이용한 구조화된 생성, 도구 호출(Tool calling) 및 iOS 26+에서의 스냅샷 스트리밍 가이드입니다.
---

# FoundationModels: 온디바이스 LLM (iOS 26)

Apple의 FoundationModels 프레임워크를 사용하여 온디바이스 언어 모델을 앱에 통합하는 패턴입니다. 텍스트 생성, `@Generable`을 이용한 구조화된 출력, 커스텀 도구 호출, 그리고 실시간 UI 업데이트를 위한 스냅샷 스트리밍을 다룹니다. 모든 작업은 개인정보 보호와 오프라인 지원을 위해 기기 내부에서 실행됩니다.

## 활성화 시점

- Apple Intelligence를 활용한 온디바이스 AI 기능 구축 시
- 클라우드 의존성 없이 텍스트 생성 또는 요약 기능 구현 시
- 자연어 입력으로부터 구조화된 데이터 추출 시
- 도메인 특화 AI 동작을 위한 커스텀 도구 호출(Tool calling) 구현 시
- 개인정보 보호가 최우선인 AI 기능 개발 시 (데이터가 기기를 벗어나지 않음)

## 주요 패턴 — 가용성 확인 (Availability Check)

세션을 생성하기 전에 항상 모델 사용 가능 여부를 먼저 확인해야 합니다.
- `.available`: 사용 가능
- `.deviceNotEligible`: 기기가 Apple Intelligence를 지원하지 않음
- `.appleIntelligenceNotEnabled`: 설정에서 활성화가 필요함
- `.modelNotReady`: 모델 다운로드 중 또는 준비되지 않음

## 기본 세션 (LanguageModelSession)

- **단일 턴(Single-turn)**: 매번 새로운 세션을 생성하여 간단한 답변 획득.
- **멀티 턴(Multi-turn)**: 대화 맥락 유지를 위해 동일한 세션 재사용.
- **지침(Instructions)**: 모델의 역할, 스타일, 안전 대책 등을 설정합니다.

## @Generable을 이용한 구조화된 생성

원시 문자열 대신 Swift 타입으로 직접 데이터를 생성합니다.
- `@Generable`: 구조화된 데이터 생성 대상임을 명시.
- `@Guide`: 숫자 범위(`.range`), 배열 개수(`.count`), 의미론적 가이드(description) 등 제약 조건 설정.
- `PartiallyGenerated`: 스트리밍 시 일부만 생성된 상태의 객체 타입.

## 도구 호출 (Tool Calling)

AI 모델이 도메인 특화 작업을 위해 앱의 커스텀 코드를 실행하도록 허용합니다.
- `Tool` 프로토콜 구현: 이름, 설명, 인자(`@Generable`) 및 실행 함수(`call`) 정의.
- `LanguageModelSession(tools:)`: 세션 생성 시 도구 목록 전달.

## 스냅샷 스트리밍 (Snapshot Streaming)

실시간 UI를 위해 구조화된 응답을 스트리밍합니다. 델타(Delta) 방식이 아닌, 각 시점의 완성된 부분적 상태(Snapshot)를 전달합니다.

## 최선 관행 (Best Practices)

- **항상 `model.availability`를 확인하십시오**: 예외 상황을 먼저 처리해야 합니다.
- **프롬프트보다 지침(Instructions)을 우선하십시오**: 모델 동작 제어에 더 효과적입니다.
- **응답 중 여부(`isResponding`)를 확인하십시오**: 세션은 한 번에 하나의 요청만 처리할 수 있습니다.
- **4,096 토큰 제한을 유의하십시오**: 지침 + 프롬프트 + 출력 합계 기준입니다. 대량 데이터는 청크로 나누어 처리하십시오.
- **결과 접근 시 `.content` 프로퍼티를 사용하십시오**: `.output`이 아닌 `.content`가 올바른 API입니다.

## 피해야 할 안티 패턴

- 가용성 확인 없이 세션 생성하기
- 4,096 토큰 맥락 창을 초과하는 입력 보내기
- 단일 세션에서 동시 요청 시도하기
- 구조화된 생성이 가능함에도 원시 문자열을 파싱하려 하는 것
- 모델이 항상 가용하다고 가정하는 것 (기기 사양 및 설정에 따라 다름)

**기억하십시오**: 온디바이스 모델은 클라우드 모델보다 빠르고 안전하지만 크기와 토큰 제한이 있습니다. 핵심 기능을 작고 집중된 작업 단위로 나누어 설계하십시오.
    
