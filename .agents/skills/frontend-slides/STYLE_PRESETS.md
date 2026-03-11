# 스타일 프리셋 참조 가이드 (Style Presets Reference)

`frontend-slides`를 위한 엄선된 시각적 스타일 모음입니다.

이 파일의 용도:
- 필수적인 뷰포트 최적화 CSS 베이스 제공
- 프리셋 선택 및 분위기 매칭 가이드
- CSS 주의사항 및 검증 규칙 안내

추상적인 형태(Abstract shapes) 위주로 사용하십시오. 사용자가 명시적으로 요청하지 않는 한 일러스트레이션은 지양하십시오.

## 뷰포트 최적화는 필수입니다

모든 슬라이드는 하나의 뷰포트에 완벽하게 들어와야 합니다.

### 황금률 (Golden Rule)

```text
각 슬라이드 = 정확히 1 뷰포트 높이 (1 Viewport Height).
콘텐츠가 너무 많으면 = 더 많은 슬라이드로 나누기.
슬라이드 내부에서 절대 스크롤하지 않기.
```

### 밀도 제한 (Density Limits)

| 슬라이드 유형 | 최대 콘텐츠 분량 |
|------------|-----------------|
| 제목 슬라이드 | 대제목 1개 + 소제목 1개 + (선택 사항) 태그라인 1개 |
| 일반 슬라이드 | 제목 1개 + 불렛 포인트 4-6개 또는 단락 2개 |
| 피처 그리드 | 최대 6개 카드 |
| 코드 슬라이드 | 최대 8-10줄 |
| 인용구 슬라이드 | 인용구 1개 + 출처 명시 |
| 이미지 슬라이드 | 이미지 1개 (가급적 60vh 이하로 제한) |

## 필수 베이스 CSS (Mandatory Base CSS)

생성되는 모든 프리젠테이션에 이 블록을 복사한 후, 그 위에 테마를 입히십시오.

```css
/* ===========================================
   뷰포트 최적화: 필수 베이스 스타일
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

## 뷰포트 점검 항목 (Viewport Checklist)

- 모든 `.slide`에 `height: 100vh`, `height: 100dvh`, `overflow: hidden`이 적용되었는가
- 모든 타이포그래피에 `clamp()`를 사용했는가
- 모든 간격 설정에 `clamp()` 또는 뷰포트 단위(vw, vh)를 사용했는가
- 이미지에 `max-height` 제한이 있는가
- 그리드 레이아웃이 `auto-fit` + `minmax()`로 유연하게 대응하는가
- `700px`, `600px`, `500px` 높이에 대한 중단점(Breakpoints)이 존재하는가
- 내용이 너무 많아 보이면 슬라이드를 나누었는가

## 분위기별 프리셋 매칭 가이드

| 분위기 | 추천 프리셋 |
|------|--------------|
| 당당하고 자신감 있는 (Impressed / Confident) | Bold Signal, Electric Studio, Dark Botanical |
| 활기차고 에너제틱한 (Excited / Energized) | Creative Voltage, Neon Cyber, Split Pastel |
| 차분하고 집중력 있는 (Calm / Focused) | Notebook Tabs, Paper & Ink, Swiss Modern |
| 영감을 주고 감동적인 (Inspired / Moved) | Dark Botanical, Vintage Editorial, Pastel Geometry |

## 프리셋 카탈로그

### 1. Bold Signal (강렬한 신호)
- 느낌: 자신감 넘치는, 파급력이 큰, 기조 연설용
- 용도: 피치 덱, 제품 출시 공지, 선언문
- 폰트: Archivo Black + Space Grotesk
- 팔레트: 차콜 베이스, 핫 오렌지색 포인트 카드, 선명한 화이트 텍스트
- 특징: 거대한 섹션 번호, 어두운 배경 위의 고대비 카드 디자인

### 2. Electric Studio (일렉트릭 스튜디오)
- 느낌: 깔끔하고 대담한, 에이전트 느낌의 세련됨
- 용도: 클라이언트 제안서, 전략 검토 보고서
- 폰트: Manrope 단일 폰트
- 팔레트: 블랙, 화이트, 채도가 높은 코발트 블루 포인트
- 특징: 2분할(Split) 패널 구성과 날카로운 에디토리얼 정렬

### 3. Creative Voltage (크리에이티브 볼티지)
- 느낌: 활동적인, 레트로-모던, 장난기 넘치는 자신감
- 용도: 크리에이티브 스튜디오, 브랜드 작업물, 제품 스토리텔링
- 폰트: Syne + Space Mono
- 팔레트: 일렉트릭 블루, 네온 옐로우, 딥 네이비
- 특징: 망점(Halftone) 텍스처, 배지(Badge) 활용, 톡톡 튀는 대비

### 4. Dark Botanical (다크 보타니컬)
- 느낌: 우아한, 프리미엄, 분위기 있는
- 용도: 럭셔리 브랜드, 깊이 있는 서사, 프리미엄 제품 소개
- 폰트: Cormorant + IBM Plex Sans
- 팔레트: 블랙에 가까운 색상, 따뜻한 아이보리, 블러쉬, 골드, 테라코타
- 특징: 흐릿한 추상 원형 오브젝트, 섬연한 선 처리, 절제된 움직임

### 5. Notebook Tabs (노트북 탭)
- 느낌: 에디토리얼(잡지풍), 정리된, 촉각을 자극하는
- 용도: 보고서, 검토 자료, 구조적인 스토리텔링
- 폰트: Bodoni Moda + DM Sans
- 팔레트: 차콜 배경 위의 크림색 종이, 파스텔 톤 탭
- 특징: 종이 시트 느낌의 배경, 컬러풀한 측면 탭, 바인더 디테일

### 6. Pastel Geometry (파스텔 지오메트리)
- 느낌: 다가가기 쉬운, 현대적인, 친근한
- 용도: 제품 개요, 온보딩 자료, 가벼운 브랜드 소개 덱
- 폰트: Plus Jakarta Sans 단일 폰트
- 팔레트: 옅은 파란색 배경, 크림색 카드, 부드러운 핑크/민트/라벤더 포인트
- 특징: 수직 형태의 필(Pill) 오브젝트, 둥근 카드 디자인, 부드러운 그림자

### 7. Split Pastel (스플릿 파스텔)
- 느낌: 장난기 넘치는, 현대적인, 창의적인
- 용도: 에이전트 소개, 워크숍, 포트폴리오
- 폰트: Outfit 단일 폰트
- 팔레트: 피치색과 라벤더색의 2분할, 민트색 배지
- 특징: 분할된 배경, 둥근 태그, 가벼운 그리드 오버레이

### 8. Vintage Editorial (빈티지 에디토리얼)
- 느낌: 재치 있는, 개성 넘치는, 잡지에서 영감을 받은
- 용도: 퍼스널 브랜드, 주관이 뚜렷한 강연, 스토리텔링
- 폰트: Fraunces + Work Sans
- 팔레트: 크림, 차콜, 더스티(Dusty) 워름 톤 포인트
- 특징: 기하학적 포인트, 테두리 있는 강조 박스, 강렬한 세리프 제목

### 9. Neon Cyber (네온 사이버)
- 느낌: 미래지향적인, 기술적인, 역동적인
- 용도: AI, 인프라, 개발 도구, 미래 기술 관련 발표
- 폰트: Clash Display + Satoshi
- 팔레트: 미드나잇 네이비, 시안(Cyan), 마젠타
- 특징: 글로우(Glow) 효과, 파티클, 그리드, 데이터 레이더 느낌의 에너지

### 10. Terminal Green (터미널 그린)
- 느낌: 개발자 중심의, 깔끔한 해커 스타일
- 용도: API, CLI 도구, 엔지니어링 데모
- 폰트: JetBrains Mono 단일 폰트
- 팔레트: GitHub 다크 테마 + 터미널 그린
- 특징: 스캔 라인, 커맨드 라인 프레임, 정교한 고정폭 폰트 리듬

### 11. Swiss Modern (스위스 모던)
- 느낌: 미니멀한, 정교한, 데이터 중심적인
- 용도: 기업용, 제품 전략, 분석 도구
- 폰트: Archivo + Nunito
- 팔레트: 화이트, 블랙, 시그널 레드
- 특징: 눈에 보이는 그리드, 비대칭성, 기하학적 규율

### 12. Paper & Ink (페이퍼 앤 잉크)
- 느낌: 문학적인, 사려 깊은, 서사 중심적인
- 용도: 에세이, 기조 연설 서사, 선언문 덱
- 폰트: Cormorant Garamond + Source Serif 4
- 팔레트: 따뜻한 크림, 차콜, 크림슨 포인트
- 특징: 풀 인용구(Pull quotes), 드롭 캡(Drop caps), 우아한 선 처리

## 직접 선택 프롬프트

사용자가 원하는 스타일을 이미 알고 있는 경우, 미리보기를 강제하지 말고 위의 프리셋 이름 중에서 직접 선택하게 하십시오.

## 애니메이션 느낌 매칭

| 느낌 | 모션 방향성 |
|---------|------------------|
| 드라마틱 / 영화 같은 | 느린 페이드, 패럴랙스, 거대한 스케일 확대 진입 |
| 기술적인 / 미래지향적인 | 글로우, 파티클, 그리드 모션, 글자 뒤섞임(Scramble) 효과 |
| 장난기 넘치는 / 친근한 | 탄력 있는 이징(Easing), 둥근 형태, 떠다니는 모션 |
| 전문적인 / 기업용 | 절제된 200-300ms 전환 효과, 깔끔한 화면 전환 |
| 차분한 / 미니멀한 | 매우 절제된 움직임, 여백 우선 디자인 |
| 에디토리얼 / 잡지 느낌 | 강력한 계층 구조, 텍스트와 이미지의 엇박자 상호작용 |

## CSS 주의사항: 함수 부정(Negating)

다음과 같이 작성하지 마십시오:

```css
right: -clamp(28px, 3.5vw, 44px);
margin-left: -min(10vw, 100px);
```

브라우저가 이를 자동으로 무시합니다.

대신 항상 다음과 같이 작성하십시오:

```css
right: calc(-1 * clamp(28px, 3.5vw, 44px));
margin-left: calc(-1 * min(10vw, 100px));
```

## 검증 대상 해상도

최소한 다음 해상도들에서 테스트하십시오:
- 데스크탑: `1920x1080`, `1440x900`, `1280x720`
- 태블릿: `1024x768`, `768x1024`
- 모바일: `375x667`, `414x896`
- 가로 모드 폰트: `667x375`, `896x414`

## 지양해야 할 안티 패턴

- 흰 배경에 보라색을 사용하는 흔한 스타트업 템플릿
- 도구적인 중립성을 의도한 경우가 아님에도 Inter / Roboto / Arial을 주력 폰트로 사용하는 것
- 불렛 포인트의 나열, 너무 작은 서체, 스크롤이 필요한 코드 블록
- 추상적인 기하학 요소가 더 효과적일 상황에서의 불필요한 장식용 일러스트레이션 사용
