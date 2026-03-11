---
name: frontend-patterns
description: React, Next.js, 상태 관리, 성능 최적화 및 UI 최선 관행(Best practices)을 위한 프론트엔드 개발 패턴 가이드입니다.
origin: ECC
---

# 프론트엔드 개발 패턴 (Frontend Development Patterns)

React, Next.js 및 고성능 사용자 인터페이스 구축을 위한 현대적인 프론트엔드 패턴입니다.

## 활성화 시점

- React 컴포넌트(합성, Props, 렌더링) 구축 시
- 상태 관리(useState, useReducer, Zustand, Context 등)가 필요할 때
- 데이터 패칭(SWR, React Query, 서버 컴포넌트 등) 구현 시
- 성능 최적화(메모이제이션, 가상화, 코드 분할)를 진행할 때
- 폼(Form) 처리 및 유효성 검사(Zod 등) 진행 시
- 접근성(Accessibility) 및 반응형 UI 패턴 구축 시

## 컴포넌트 패턴

### 1. 상속보다 합성 (Composition Over Inheritance)
상속 대신 컴포넌트를 조합하여 재사용성을 높입니다. `children`을 활용하십시오.

### 2. 복합 컴포넌트 (Compound Components)
하나의 상태를 공유하는 여러 컴포넌트를 만들어 유연한 UI를 구성합니다. (예: `Tabs`, `TabList`, `Tab`)

### 3. Render Props 패턴
컴포넌트의 자식으로 함수를 전달하여 렌더링 로직을 외부에서 제어하게 합니다.

## 커스텀 훅(Custom Hooks) 패턴

- **상태 관리 훅**: `useToggle` 등 자주 반복되는 상태 로직 캡슐화.
- **비동기 데이터 패칭 훅**: `useQuery`와 같이 로딩, 에러, 데이터를 통합 관리하는 패턴.
- **디바운스(Debounce) 훅**: 입력값의 빈번한 변화를 지연 처리.

## 상태 관리 패턴

- **Context + Reducer**: 전역 상태와 복잡한 업데이트 로직을 처리하는 공식 패턴.
- **Zustand / Recoil**: 더 가볍거나 아토믹한 상태 관리가 필요할 때 사용하십시오.

## 성능 최적화 (Performance Optimization)

- **메모이제이션**: `useMemo`, `useCallback`, `React.memo`를 적절히 사용하여 불필요한 재렌더링을 방지하십시오.
- **코드 분할 및 지연 로딩**: `lazy`, `Suspense`를 사용하여 무거운 컴포넌트를 필요할 때 로드하십시오.
- **가상화(Virtualization)**: 대량의 목록을 렌더링할 때 화면에 보이는 부분만 렌더링하여 DOM 부하를 줄이십시오.

## 폼 핸들링 패턴

- 제어 컴포넌트(Controlled components)를 기본으로 하며, 복잡한 검증은 `Zod` 스키마와 통합하십시오.

## 에러 바운더리 (Error Boundary)

클래스 컴포넌트를 사용하여 하위 트리에서 발생하는 에러를 포착하고 사용자 정의 UI를 보여주는 패턴입니다.

## 애니메이션 및 접근성

- **Framer Motion**: 리스트나 모달에 자연스러운 애니메이션 적용.
- **키보드 내비게이션**: 드롭다운, 모달 등에서 화살표 키와 Enter, Escape 키 처리를 보장하십시오.
- **포커스 관리**: 모달이 열릴 때 포커스를 이동시키고 닫힐 때 원래 위치로 복원하십시오.

**기억하십시오**: 현대적인 프론트엔드 패턴은 유지보수가 쉽고 성능이 뛰어난 UI 개발을 가능하게 합니다. 프로젝트의 복잡도에 맞는 적절한 패턴을 선택하십시오.
    
