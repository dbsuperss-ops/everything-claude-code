---
name: frontend-slides-style-presets
description: frontend-slides를 위한 시각적 스타일 프리셋 및 디자인 가이드라인입니다. 뷰포트 최적화 CSS, 분위기별 스타일 매핑, CSS 주의사항 등을 포함합니다.
origin: ECC
---

# 스타일 프리셋 참고 가이드

`frontend-slides` 제작 시 활용할 수 있는 시각적 스타일 가이드입니다.

이 파일은 다음 용도로 사용하십시오:
* **강제적인 뷰포트 최적화 CSS 기초 설정**
* **분위기(Mood)에 따른 스타일 프리셋 선택 및 매핑**
* **CSS 작성 시 주의사항 및 검증 규칙 확인**

*디자인 원칙: 특별한 요청이 없는 한 일러스트레이션보다는 추상적인 도형과 기하학적 패턴을 우선적으로 사용하십시오.*

## 뷰포트 최적화 (Viewport-fit) 원칙

모든 슬라이드는 반드시 하나의 화면(Viewport)에 완벽히 들어맞아야 합니다.

### 황금률
* **하나의 슬라이드 = 정확히 하나의 뷰포트 높이**
* **내용이 너무 많음 = 슬라이드를 여러 개로 분할**
* **슬라이드 내부 스크롤 = 절대 엄금**

### 콘텐츠 밀도 제한
| 슬라이드 유형 | 최대 권장량 |
|---|---|
| 제목 슬라이드 | 메인 제목 1 + 부제목 1 + (선택) 태그라인 |
| 내용 슬라이드 | 제목 1 + 불렛포인트 4~6개 또는 단락 2개 |
| 그리드/카드 | 최대 6개 카드 |
| 코드 슬라이드 | 최대 8~10줄 |
| 인용 슬라이드 | 인용구 1 + 출처 |
| 이미지 슬라이드 | 이미지 1장 (높이 60vh 이하 권장) |

## 필수 기본 CSS (Mandatory Base CSS)

모든 발표 자료 생성 시 이 코드 블록을 기초로 삼고, 그 위에 테마를 적용하십시오.

```css
/* ===========================================
   VIEWPORT FITTING: 필수 기본 스타일
   =========================================== */

html, body {
    height: 100%;
    overflow-x: hidden;
    margin: 0;
    padding: 0;
}

html {
    scroll-snap-type: y mandatory;
    scroll-behavior: smooth;
}

.slide {
    width: 100vw;
    height: 100vh;
    height: 100dvh;
    overflow: hidden;
    scroll-snap-align: start;
    display: flex;
    flex-direction: column;
    position: relative;
}

.slide-content {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    max-height: 100%;
    overflow: hidden;
    padding: var(--slide-padding);
}

:root {
    /* 화면 크기에 반응하는 가변 폰트 크기 */
    --title-size: clamp(1.5rem, 5vw, 4rem);
    --h2-size: clamp(1.25rem, 3.5vw, 2.5rem);
    --body-size: clamp(0.75rem, 1.5vw, 1.125rem);

    /* 가변 간격 설정 */
    --slide-padding: clamp(1rem, 4vw, 4rem);
    --content-gap: clamp(0.5rem, 2vw, 2rem);
}

/* 요소 크기 제한 */
.card, .container {
    max-width: min(90vw, 1000px);
    max-height: min(80vh, 700px);
}

/* 미디어 쿼리: 작은 화면(높이가 낮은 화면) 대응 */
@media (max-height: 600px) {
    :root {
        --slide-padding: clamp(0.5rem, 2.5vw, 1.5rem);
        --title-size: clamp(1.1rem, 4vw, 2rem);
    }
    .nav-dots, .keyboard-hint { display: none; }
}

/* 애니메이션 줄이기 설정 준수 */
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.2s !important;
    }
}
```

## 스타일 프리셋 카탈로그

### 1. Bold Signal
* **분위기**: 자신감, 강력한 임팩트, 키노트 스타일
* **추천 용도**: 투자 피칭, 제품 런칭, 중대 발표
* **폰트**: Archivo Black + Space Grotesk
* **색상**: 차콜 베이스 + 밝은 오렌지 포인트 + 순백색 텍스트

### 2. Electric Studio
* **분위기**: 깔끔함, 대담함, 전문적인 세련미
* **추천 용도**: 클라이언트 미팅, 전략 리뷰
* **폰트**: Manrope 단일 폰트 계층 활용
* **색상**: 블랙 & 화이트 + 채도 높은 코발트 블루 포인트

### 3. Dark Botanical
* **분위기**: 우아함, 고혹적임, 감성적임
* **추천 용도**: 럭셔리 브랜드, 깊이 있는 서사, 고급 제품 소개
* **폰트**: Cormorant + IBM Plex Sans
* **색상**: 거의 검은색에 가까운 배경 + 웜 아이보리/골드/테라코타 포인트

### 4. Neon Cyber
* **분위기**: 미래지향적, 기술 중심적, 역동적
* **추천 용도**: AI, 인프라, 개발 도구, 미래 트렌드 강연
* **폰트**: Clash Display + Satoshi
* **색상**: 미드나잇 네이비 + 시안(Cyan) + 마젠타 레이저 포인트

### 5. Terminal Green
* **분위기**: 개발자 친화적, 해커 스타일, 미니멀리즘
* **추천 용도**: API 소개, CLI 도구, 엔지니어링 브리핑
* **폰트**: JetBrains Mono 전용
* **색상**: GitHub 다크 테마 배경 + 터미널 그린 색상

## 분위기별 애니메이션 매핑
| 분위기 | 모션 방향성 |
|---|---|
| **드라마틱 / 시네마틱** | 느린 페이드 인/아웃, 패럴랙스 스크롤, 대담한 스케일 변화 |
| **테크 / 미래적** | 글리치 효과, 입자 애니메이션, 텍스트가 섞여 나타나는 효과 |
| **장난기 / 친근함** | 바운스 효과, 유동적인 곡선 움직임, 떠다니는 듯한 모션 |
| **전문적 / 기업형** | 짧고 간결한(200-300ms) 전환, 깔끔한 슬라이드 교체 |

## CSS 주의사항: 음수 값 처리
`clamp()`나 `min()` 함수 앞에 직접 마이너스(`-`) 부호를 붙이면 브라우저가 무시할 수 있습니다.
* **나쁨**: `right: -clamp(20px, 5vw, 40px);`
* **좋음**: `right: calc(-1 * clamp(20px, 5vw, 40px));`

## 검증 대상 해상도
* **데스크톱**: 1920x1080 (FHD), 1440x900
* **태블릿**: 1024x768 (가로), 768x1024 (세로)
* **모바일**: 375x667, 414x896 (세로 및 가로 모드 모두 확인)

**안티 패턴 경고**: 
* 내용이 너무 많아 폰트가 지나치게 작아지는 경우 (슬라이드 분할 필수)
* 가독성이 떨어지는 색상 조합 (대비 확인 필수)
* 스크롤이 생기는 코드 블록 또는 텍스트 박스
* 단순히 장식용으로만 쓰이는 의미 없는 일러스트 남용
