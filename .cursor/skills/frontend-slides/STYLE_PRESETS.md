# 스타일 프리셋 참조 (Style Presets Reference)

`frontend-slides`를 위해 선별된 시각적 스타일 가이드입니다.

이 파일의 용도:
- 필수적인 뷰포트 맞춤 CSS 베이스 확인
- 프리셋 선택 및 분위기 매핑
- CSS 주의 사항 및 검증 규칙 확인

추상적인 도형 위주로 사용하십시오. 사용자가 명시적으로 요청하지 않는 한 일러스트레이션은 지양하십시오.

## 뷰포트 맞춤은 필수입니다 (Non-Negotiable)

모든 슬라이드는 하나의 뷰포트에 완전히 들어와야 합니다.

### 황금률 (Golden Rule)

```text
각 슬라이드 = 정확히 1 뷰포트 높이.
콘텐츠가 너무 많음 = 슬라이드를 더 나눔.
슬라이드 내부에서 절대 스크롤하지 않음.
```

### 밀도 제한 (Density Limits)

| 슬라이드 유형 | 최대 콘텐츠 제한 |
|------------|-----------------|
| 제목 슬라이드 | 헤딩 1개 + 부제목 1개 + (선택) 태그라인 |
| 일반 슬라이드 | 헤딩 1개 + 불렛 포인트 4~6개 또는 단락 2개 |
| 특징 그리드 | 최대 카드 6개 |
| 코드 슬라이드 | 최대 8~10줄 |
| 인용구 슬라이드 | 인용문 1개 + 출처 |
| 이미지 슬라이드 | 이미지 1개, 가급적 60vh 이하 권장 |

## 필수 베이스 CSS

생성되는 모든 프레젠테이션에 이 블록을 복사한 뒤, 그 위에 테마를 입히십시오.

```css
/* ===========================================
   VIEWPORT FITTING: MANDATORY BASE STYLES
   =========================================== */

html, body {
    height: 100%;
    overflow-x: hidden;
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
    --title-size: clamp(1.5rem, 5vw, 4rem);
    --h2-size: clamp(1.25rem, 3.5vw, 2.5rem);
    --h3-size: clamp(1rem, 2.5vw, 1.75rem);
    --body-size: clamp(0.75rem, 1.5vw, 1.125rem);
    --small-size: clamp(0.65rem, 1vw, 0.875rem);

    --slide-padding: clamp(1rem, 4vw, 4rem);
    --content-gap: clamp(0.5rem, 2vw, 2rem);
    --element-gap: clamp(0.25rem, 1vw, 1rem);
}

.card, .container, .content-box {
    max-width: min(90vw, 1000px);
    max-height: min(80vh, 700px);
}

.feature-list, .bullet-list {
    gap: clamp(0.4rem, 1vh, 1rem);
}

.feature-list li, .bullet-list li {
    font-size: var(--body-size);
    line-height: 1.4;
}

.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(100%, 250px), 1fr));
    gap: clamp(0.5rem, 1.5vw, 1rem);
}

img, .image-container {
    max-width: 100%;
    max-height: min(50vh, 400px);
    object-fit: contain;
}

@media (max-height: 700px) {
    :root {
        --slide-padding: clamp(0.75rem, 3vw, 2rem);
        --content-gap: clamp(0.4rem, 1.5vw, 1rem);
        --title-size: clamp(1.25rem, 4.5vw, 2.5rem);
        --h2-size: clamp(1rem, 3vw, 1.75rem);
    }
}

@media (max-height: 600px) {
    :root {
        --slide-padding: clamp(0.5rem, 2.5vw, 1.5rem);
        --content-gap: clamp(0.3rem, 1vw, 0.75rem);
        --title-size: clamp(1.1rem, 4vw, 2rem);
        --body-size: clamp(0.7rem, 1.2vw, 0.95rem);
    }

    .nav-dots, .keyboard-hint, .decorative {
        display: none;
    }
}

@media (max-height: 500px) {
    :root {
        --slide-padding: clamp(0.4rem, 2vw, 1rem);
        --title-size: clamp(1rem, 3.5vw, 1.5rem);
        --h2-size: clamp(0.9rem, 2.5vw, 1.25rem);
        --body-size: clamp(0.65rem, 1vw, 0.85rem);
    }
}

@media (max-width: 600px) {
    :root {
        --title-size: clamp(1.25rem, 7vw, 2.5rem);
    }

    .grid {
        grid-template-columns: 1fr;
    }
}

@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.2s !important;
    }

    html {
        scroll-behavior: auto;
    }
}
```

## 뷰포트 체크리스트

- 모든 `.slide`가 `height: 100vh`, `height: 100dvh`, `overflow: hidden`을 가지고 있는가
- 모든 타이포그래피가 `clamp()`를 사용하고 있는가
- 모든 간격이 `clamp()` 또는 뷰포트 단위를 사용하고 있는가
- 이미지에 `max-height` 제약 조건이 있는가
- 그리드가 `auto-fit` + `minmax()`로 유동적으로 변하는가
- `700px`, `600px`, `500px` 높이에 대한 브레이크포인트가 존재하는가
- 슬라이드가 다소 답답하게 느껴진다면 슬라이드를 나누었는가

## 분위기에 따른 프리셋 매핑

| 분위기 | 추천 프리셋 |
|------|--------------|
| 인상적인 / 자신감 있는 | Bold Signal, Electric Studio, Dark Botanical |
| 흥분되는 / 활기찬 | Creative Voltage, Neon Cyber, Split Pastel |
| 차분한 / 집중되는 | Notebook Tabs, Paper & Ink, Swiss Modern |
| 영감을 주는 / 감동적인 | Dark Botanical, Vintage Editorial, Pastel Geometry |

## 프리셋 카탈로그 (Catalog)

### 1. Bold Signal
- 분위기: 자신감 있는, 강렬한 임팩트, 기조연설 레디
- 용도: 피치 덱, 출시 발표, 선언문
- 폰트: Archivo Black + Space Grotesk
- 색상: 차콜 베이스, 오렌지 포인트 카드, 선명한 화이트 텍스트
- 특징: 거대한 섹션 번호, 어두운 배경 위 고대비 카드

### 2. Electric Studio
- 분위기: 깔끔한, 대담한, 에이전트 느낌의 세련미
- 용도: 고객 프레젠테이션, 전략 리뷰
- 폰트: Manrope 단일 폰트
- 색상: 블랙, 화이트, 채도 높은 코발트 블루 포인트
- 특징: 2단 패널 분할과 날카로운 편집용 정렬

### 3. Creative Voltage
- 분위기: 에너지 넘치는, 레트로 모던, 위트 있는 자신감
- 용도: 크리에이티브 스튜디오, 브랜드 작업, 제품 스토리텔링
- 폰트: Syne + Space Mono
- 색상: 일렉트릭 블루, 네온 옐로우, 딥 네이비
- 특징: 망점(Halftone) 질감, 배지, 톡톡 튀는 대비

### 4. Dark Botanical
- 분위기: 우아한, 프리미엄, 분위기 있는
- 용도: 럭셔리 브랜드, 사색적인 서사, 프리미엄 제품 소개
- 폰트: Cormorant + IBM Plex Sans
- 색상: 블랙에 가까운 차콜, 웜 아이보리, 블러쉬, 골드, 테라코타
- 특징: 흐릿한 추상적 원형, 섬세한 라인, 절제된 모션

### 5. Notebook Tabs
- 분위기: 잡지 느낌의, 정리된, 촉각적인
- 용도: 보고서, 리뷰, 구조화된 스토리텔링
- 폰트: Bodoni Moda + DM Sans
- 색상: 차콜 위 크림색 종이와 파스텔톤 탭
- 특징: 종이 질감, 유색 사이드 탭, 바인더 디테일

### 6. Pastel Geometry
- 분위기: 친근한, 현대적인, 부드러운
- 용도: 제품 개요, 온보딩, 가벼운 브랜드 덱
- 폰트: Plus Jakarta Sans 단일 폰트
- 색상: 옅은 블루 배경, 크림 카드, 소프트 핑크/민트/라벤더 포인트
- 특징: 수직 캡슐 형태, 둥근 카드, 부드러운 그림자

### 7. Split Pastel
- 분위기: 위트 있는, 현대적인, 창의적인
- 용도: 에이전트 소개, 워크숍, 포트폴리오
- 폰트: Outfit 단일 폰트
- 색상: 피치 + 라벤더 분할 배경과 민트 배지
- 특징: 분할된 배경, 둥근 태그, 가벼운 그리드 오버레이

### 8. Vintage Editorial
- 분위기: 재치 있는, 개성이 강한, 매거진 풍
- 용도: 퍼스널 브랜드, 의견 주도형 발표, 스토리텔링
- 폰트: Fraunces + Work Sans
- 색상: 크림, 차콜, 더스티 웜 포인트
- 특징: 기하학적 포인트, 테두리가 있는 강조 박스, 강렬한 세리프 헤드라인

### 9. Neon Cyber
- 분위기: 미래지향적인, 테크니컬한, 역동적인
- 용도: AI, 인프라, 개발자 도구, 'Future-of-X' 관련 발표
- 폰트: Clash Display + Satoshi
- 색상: 미드나잇 네이비, 시안, 마젠타
- 특징: 글로우 효과, 파티클, 그리드, 데이터 레이더 에너지

### 10. Terminal Green
- 분위기: 개발자 중심의, 해커 스타일의 깔끔함
- 용도: API, CLI 도구, 엔지니어링 데모
- 폰트: JetBrains Mono 단일 폰트
- 색상: GitHub 다크 모드 색상 + 터미널 그린
- 특징: 스캔 라인, 커맨드 라인 프레임, 정밀한 고정폭 리듬

### 11. Swiss Modern
- 분위기: 미니멀한, 정밀한, 데이터 중심적인
- 용도: 기업 발표, 제품 전략, 분석 자료
- 폰트: Archivo + Nunito
- 색상: 화이트, 블랙, 시그널 레드
- 특징: 눈에 보이는 그리드, 비대칭성, 기하학적 규율

### 12. Paper & Ink
- 분위기: 문학적인, 사색적인, 서사 중심적인
- 용도: 에세이, 기조연설 서사, 매니페스토 덱
- 폰트: Cormorant Garamond + Source Serif 4
- 색상: 따뜻한 크림, 차콜, 크림슨 포인트
- 특징: 인용구 강조, 드롭 캡(첫 글자 강조), 우아한 구분선

## 직접 선택 프롬프트

사용자가 이미 원하는 스타일을 알고 있는 경우에는 프리뷰 생성을 강요하지 말고 위의 프리셋 이름 중에서 직접 선택하게 하십시오.

## 애니메이션 느낌 매핑

| 느낌 | 모션 방향성 |
|---------|------------------|
| 드라마틱 / 시네마틱 | 느린 페이드, 패럴랙스, 큰 스케일 변화(Scale-in) |
| 테크니컬 / 미래적 | 글로우, 파티클, 그리드 모션, 텍스트 스크램블 |
| 장난스러운 / 친근한 | 탄성 있는 이징(Easing), 둥근 도형, 떠다니는 모션 |
| 전문적인 / 기업용 | 절제된 200~300ms 트랜지션, 정갈한 슬라이드 |
| 차분한 / 미니멀 | 극도로 절제된 움직임, 여백 중심 |
| 편집 디자인 / 매거진 | 강한 계층 구조, 텍스트와 이미지의 엇갈린 교차 |

## CSS 주의 사항: 함수 무효화

다음과 같이 작성하지 마십시오:

```css
right: -clamp(28px, 3.5vw, 44px);
margin-left: -min(10vw, 100px);
```

브라우저가 이를 무시합니다.

대신 항상 다음과 같이 작성하십시오:

```css
right: calc(-1 * clamp(28px, 3.5vw, 44px));
margin-left: calc(-1 * min(10vw, 100px));
```

## 검증 해상도

최소한 다음 해상도에서 테스트하십시오:
- 데스크톱: `1920x1080`, `1440x900`, `1280x720`
- 태블릿: `1024x768`, `768x1024`
- 모바일: `375x667`, `414x896`
- 가로형 폰트: `667x375`, `896x414`

## 안티 패턴 (지양 사항)

- 시각적 정체성 없는 보라색 배경+흰색 텍스트 스타트업 템플릿
- 유틸리티적인 중립성을 의도한 경우가 아님에도 Inter / Roboto / Arial 사용
- 불렛 포인트 나열, 작은 글씨, 스크롤이 필요한 코드 블록
- 추상적 기하학 요소가 더 적합한 상황에서의 장식용 일러스트레이션 사용
