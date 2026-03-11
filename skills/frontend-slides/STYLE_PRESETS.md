# 스타일 프리셋 레퍼런스 (Style Presets Reference)

`frontend-slides`를 위해 엄선된 비주얼 스타일 가이드입니다.

이 문서는 다음을 위해 사용하십시오:
- 필수적인 뷰포트 맞춤(Viewport-fitting) CSS 베이스 설정
- 프리셋 선택 및 분위기 매핑 (Mood mapping)
- CSS 주의 사항 및 검증 규칙 확인

추상적인 형태만 사용하십시오. 사용자가 명시적으로 요청하지 않는 한 일러스트레이션은 지양하십시오.

## 뷰포트 맞춤 (Viewport Fit) — 필수 사항

모든 슬라이드는 하나의 뷰포트에 완전히 들어차야 합니다.

### 황금률 (Golden Rule)

```text
각 슬라이드 = 정확히 하나의 뷰포트 높이
콘텐츠가 너무 많을 경우 = 슬라이드를 여러 개로 분할
슬라이드 내부에서 스크롤 금지
```

### 밀도 제한 (Density Limits)

| 슬라이드 유형 | 최대 콘텐츠 양 |
|------------|-----------------|
| 제목 슬라이드 | 제목 1 + 부제목 1 + (선택 사항) 태그라인 |
| 콘텐츠 슬라이드 | 제목 1 + 불렛 포인트 4-6개 또는 문단 2개 |
| 피처 그리드 | 최대 6개 카드 |
| 코드 슬라이드 | 최대 8-10줄 |
| 인용 슬라이드 | 인용구 1 + 출처 |
| 이미지 슬라이드 | 이미지 1개, 가급적 60vh 이하 권장 |

## 필수 베이스 CSS

생성되는 모든 프레젠테이션에 이 블록을 복사한 뒤, 그 위에 테마를 입히십시오.

```css
/* ===========================================
   뷰포트 맞춤: 필수 베이스 스타일
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
- 모든 타이포그래피에 `clamp()`를 사용했는가
- 모든 간격 설정에 `clamp()` 또는 뷰포트 단위를 사용했는가
- 이미지에 `max-height` 제한이 걸려 있는가
- 그리드가 `auto-fit` + `minmax()`를 사용하여 가변적으로 대응하는가
- `700px`, `600px`, `500px` 높이에 대한 브레이크포인트가 존재하는가
- 슬라이드가 답답하게 느껴진다면, 슬라이드를 분할했는가

## 분위기별 프리셋 매핑

| 분위기 | 추천 프리셋 |
|------|--------------|
| 인상적임 / 자신감 있음 | Bold Signal, Electric Studio, Dark Botanical |
| 신남 / 에너제틱함 | Creative Voltage, Neon Cyber, Split Pastel |
| 차분함 / 집중력 있음 | Notebook Tabs, Paper & Ink, Swiss Modern |
| 영감을 줌 / 감동적임 | Dark Botanical, Vintage Editorial, Pastel Geometry |

## 프리셋 카탈로그

### 1. Bold Signal
- 느낌: 자신감 넘치는, 강력한 임팩트, 기조연설(Keynote) 스타일
- 위 용도: 피치 덱, 런칭 행사, 선언문
- 폰트: Archivo Black + Space Grotesk
- 팔레트: 차콜 베이스, 핫 오렌지 컬러의 핵심 카드, 깔끔한 화이트 텍스트
- 특징: 거대한 섹션 번호, 어두운 배경 위의 고대비 카드

### 2. Electric Studio
- 느낌: 깔끔하고 대담한, 에이전트 느낌의 세련됨
- 위 용도: 클라이언트 프레젠테이션, 전략 리뷰
- 폰트: Manrope 단일 폰트
- 팔레트: 블랙, 화이트, 채도 높은 코발트 블루 강조색
- 특징: 2단 분할 패널 및 날카로운 편집용 정렬 스타일

### 3. Creative Voltage
- 느낌: 활기찬, 레트로-모던, 장난스러운 자신감
- 위 용도: 크리에이티브 스튜디오, 브랜드 작업, 제품 스토리텔링
- 폰트: Syne + Space Mono
- 팔레트: 일렉트릭 블루, 네온 옐로우, 깊은 네이비
- 특징: 하프톤 텍스처, 배지 장식, 강렬한 대비

### 4. Dark Botanical
- 느낌: 우아한, 프리미엄, 감각적인 분위기
- 위 용도: 럭셔리 브랜드, 사려 깊은 내러티브, 프리미엄 제품 소개
- 폰트: Cormorant + IBM Plex Sans
- 팔레트: 블랙에 가까운 색상, 따뜻한 아이보리, 블러시 핑크, 골드, 테라코타
- 특징: 블러 처리된 추상적인 원형 패턴, 가느다란 가로줄(Rule), 절제된 모션

### 5. Notebook Tabs
- 느낌: 에디토리얼 스타일, 체계적인, 촉각적인
- 위 용도: 보고서, 리뷰, 구조화된 스토리텔링
- 폰트: Bodoni Moda + DM Sans
- 팔레트: 차콜 배경 위의 크림 색상지, 파스텔 톤 탭
- 특징: 종이 질감, 유색 사이드 탭, 바인더 디테일

### 6. Pastel Geometry
- 느낌: 접근하기 쉬운, 현대적인, 친근한
- 위 용도: 제품 개요, 온보딩, 가벼운 브랜드 소개
- 폰트: Plus Jakarta Sans 단일 폰트
- 팔레트: 연한 블루 배경, 크림색 카드, 부드러운 핑크/민트/라벤더 포인트
- 특징: 수직 필(Pill) 형태, 둥근 카드, 부드러운 그림자

### 7. Split Pastel
- 느낌: 장난기 가득한, 현대적인, 창의적인
- 위 용도: 에이전시 소개, 워크숍, 포트폴리오
- 폰트: Outfit 단일 폰트
- 팔레트: 피치 + 라벤더 분할 배경, 민트색 배지
- 특징: 분할된 배경화면, 둥근 태그, 가벼운 그리드 오버레이

### 8. Vintage Editorial
- 느낌: 위트 있는, 개성 넘치는, 잡지에서 영감을 받은
- 위 용도: 퍼스널 브랜드, 소신 있는 강연, 스토리텔링
- 폰트: Fraunces + Work Sans
- 팔레트: 크림, 차콜, 톤다운된 따뜻한 포인트 컬러
- 특징: 기하학적 포인트, 테두리가 있는 강조 박스(Callouts), 강렬한 세리프 헤드라인

### 9. Neon Cyber
- 느낌: 미래지향적인, 테크니컬한, 운동감 있는
- 위 용도: AI, 인프라, 개발 도구, 미래 담론
- 폰트: Clash Display + Satoshi
- 팔레트: 미드나잇 네이비, 시안(Cyan), 마젠타(Magenta)
- 특징: 광채 효과(Glow), 입자(Particles), 그리드 패턴, 데이터 레이더 느낌

### 10. Terminal Green
- 느낌: 개발자 중심의, 해커 스타일의 깔끔함
- 위 용도: API, CLI 도구, 엔지니어링 데모
- 폰트: JetBrains Mono 단일 폰트
- 팔레트: GitHub 다크 모드 스타일 + 터미널 그린
- 특징: 스캔 라인, 커맨드 라인 프레임, 정밀한 고정 폭(Monospace) 리듬

### 11. Swiss Modern
- 느낌: 미니멀한, 정밀한, 데이터 중심적인
- 위 용도: 기업 IR, 제품 전략, 분석 도구
- 폰트: Archivo + Nunito
- 팔레트: 화이트, 블랙, 시그널 레드
- 특징: 가시적인 그리드 라인, 비대칭성, 기하학적 규율

### 12. Paper & Ink
- 느낌: 문학적인, 사려 깊은, 스토리 중심의
- 위 용도: 에세이, 기조연설 내러티브, 매니페스토(Manifesto) 덱
- 폰트: Cormorant Garamond + Source Serif 4
- 팔레트: 따뜻한 크림색, 차콜, 크림슨 레드 포인트
- 특징: 인용문(Pull quotes), 드롭 캡(Drop caps), 우아한 가로줄

## 직접 선택 프롬프트

사용자가 이미 원하는 스타일을 알고 있다면, 미리보기 생성을 강요하지 말고 위의 프리셋 이름 중에서 직접 선택할 수 있도록 하십시오.

## 애니메이션 느낌 매핑

| 느낌 | 모션 방향 |
|---------|------------------|
| 드라마틱함 / 영화 같음 | 느린 페이드, 패럴랙스(Parallax), 거대한 스케일-인 |
| 테크니컬함 / 미래지향적임 | 광채 효과, 입자 효과, 그리드 모션, 스크램블 텍스트 |
| 장난기 있음 / 친근함 | 탄력 있는(Springy) 이징, 둥근 형태, 부유하는 듯한 모션 |
| 프로페셔널함 / 기업 스타일 | 절제된 200-300ms 트랜지션, 깔끔한 슬라이드 전환 |
| 차분함 / 미니멀함 | 매우 절제된 움직임, 여백 중심의 레이아웃 |
| 에디토리얼 / 잡지 스타일 | 강한 위계 구조, 텍스트와 이미지의 시차를 둔 배치 |

## CSS 주의 사항: 부정 함수 (Negating Functions)

절대로 다음과 같이 작성하지 마십시오:

```css
right: -clamp(28px, 3.5vw, 44px);
margin-left: -min(10vw, 100px);
```

브라우저는 이를 무시합니다.

대신 항상 다음과 같이 작성하십시오:

```css
right: calc(-1 * clamp(28px, 3.5vw, 44px));
margin-left: calc(-1 * min(10vw, 100px));
```

## 검증 사이즈

최소한 다음 해상도에서 테스트하십시오:
- 데스크톱: `1920x1080`, `1440x900`, `1280x720`
- 태블릿: `1024x768`, `768x1024`
- 모바일: `375x667`, `414x896`
- 가로형 휴대폰: `667x375`, `896x414`

## 피해야 할 안티 패턴

다음을 사용하지 마십시오:
- 전형적인 "화이트 배경에 퍼플" 스타트업 템플릿
- 사용자가 명시적으로 중립적인 스타일을 원하지 않는 한 Inter / Roboto / Arial 사용 자제
- 텍스트로 가득 찬 불렛 포인트 나열, 너무 작은 글씨, 스크롤이 필요한 코드 블록
- 추상 기하학적 문양으로 충분한 곳에 불필요한 일러스트레이션 삽입
