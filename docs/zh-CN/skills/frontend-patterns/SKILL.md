---
name: frontend-patterns
description: React, Next.js, 상태 관리, 성능 최적화 및 UI 베스트 프랙티스를 위한 현대적 프론트엔드 개발 패턴 가이드입니다.
origin: ECC
---

# 프론트엔드 개발 패턴

React, Next.js 및 고성능 사용자 인터페이스(UI) 구축을 위한 현대적인 프론트엔드 패턴입니다.

## 적용 시점

* React 컴포넌트를 설계할 때 (컴포지션, Props, 렌더링 전략)
* 상태 관리를 구현할 때 (useState, useReducer, Zustand, Context API 등)
* 데이터 페칭(Data Fetching) 로직을 작성할 때 (SWR, React Query, Server Components)
* 성능 최적화가 필요할 때 (Memoization, Virtualization, Code Splitting)
* 복잡한 폼(Form)을 처리할 때 (유효성 검사, Controlled inputs, Zod schema)
* 클라이언트 사이드 라우팅 및 네비게이션을 구현할 때
* 접근성(Accessibility)과 반응형 UI 패턴을 구축할 때

## 컴포넌트 패턴

### 상속보다 합성 (Composition over Inheritance)

```typescript
// ✅ 권장: 컴포넌트 합성 방식
interface CardProps {
  children: React.ReactNode
  variant?: 'default' | 'outlined'
}

export function Card({ children, variant = 'default' }: CardProps) {
  return <div className={`card card-${variant}`}>{children}</div>
}

export function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>
}

export function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>
}

// 사용 예시
<Card>
  <CardHeader>제목</CardHeader>
  <CardBody>컨텐츠 내용</CardBody>
</Card>
```

### 복합 컴포넌트 (Compound Components)

```typescript
// 내부 상태를 공유하는 관련 컴포넌트 그룹 구성
export function Tabs({ children, defaultTab }: { children: React.ReactNode, defaultTab: string }) {
  const [activeTab, setActiveTab] = useState(defaultTab)
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  )
}

export function Tab({ id, children }: { id: string, children: React.ReactNode }) {
  const context = useContext(TabsContext)
  return (
    <button 
      className={context.activeTab === id ? 'active' : ''} 
      onClick={() => context.setActiveTab(id)}
    >
      {children}
    </button>
  )
}
```

## 커스텀 훅 (Custom Hooks) 패턴

### 상태 제어 훅 (State Management)
```typescript
export function useToggle(initialValue = false): [boolean, () => void] {
  const [value, setValue] = useState(initialValue)
  const toggle = useCallback(() => setValue(v => !v), [])
  return [value, toggle]
}
```

### 비동기 데이터 페칭 훅
```typescript
export function useQuery<T>(key: string, fetcher: () => Promise<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(false)
  
  const refetch = useCallback(async () => {
    setLoading(true)
    try {
      const result = await fetcher()
      setData(result)
    } finally {
      setLoading(false)
    }
  }, [fetcher])

  useEffect(() => { refetch() }, [key, refetch])
  return { data, loading, refetch }
}
```

## 상태 관리 패턴

### Context + Reducer 패턴 (복잡한 로컬 상태)
```typescript
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_DATA': return { ...state, data: action.payload }
    default: return state
  }
}

export function DataProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <DataContext.Provider value={{ state, dispatch }}>
      {children}
    </DataContext.Provider>
  )
}
```

## 성능 최적화

### 메모이제이션 (Memoization)
```typescript
// ✅ 비용이 큰 계산 결과 재사용
const sortedData = useMemo(() => data.sort(), [data])

// ✅ 자식 컴포넌트에 전달되는 함수 고정
const handleClick = useCallback(() => { /* ... */ }, [])

// ✅ 순수 컴포넌트 렌더링 방지
export const ListItem = React.memo(({ item }) => <div>{item.name}</div>)
```

### 코드 분할 및 지연 로딩 (Lazy Loading)
```typescript
import { lazy, Suspense } from 'react'

const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<LoadingSkeleton />}>
      <HeavyChart />
    </Suspense>
  )
}
```

## 표마(Form) 처리 패턴

### 검증을 포함한 제어 컴포넌트
```typescript
export function RegistrationForm() {
  const [values, setValues] = useState({ email: '', password: '' })
  const [errors, setErrors] = useState({})

  const validate = () => {
    const newErrors = {}
    if (!values.email.includes('@')) newErrors.email = '유효한 이메일을 입력하세요'
    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }

  const handleSubmit = (e) => {
    e.preventDefault()
    if (validate()) { /* 서버 전송 */ }
  }
}
```

## 애니메이션 패턴 (Framer Motion)
```typescript
import { motion, AnimatePresence } from 'framer-motion'

export function FadeInList({ items }) {
  return (
    <AnimatePresence>
      {items.map(item => (
        <motion.div
          key={item.id}
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
        >
          {item.content}
        </motion.div>
      ))}
    </AnimatePresence>
  )
}
```

## 웹 접근성 (A11y)

### 키보드 내비게이션 및 포커스 관리
* **로킹(Trap Focus)**: 모달이 열려 있을 때 포커스가 모달 밖으로 나가지 않도록 관리하십시오.
* **ARIA 속성**: `role="dialog"`, `aria-modal="true"`, `aria-label` 등을 적절히 사용하여 스크린 리더 사용자를 배려하십시오.
* **Skip Link**: 반복되는 내비게이션을 건너뛰고 본문으로 바로 이동할 수 있는 링크를 제공하십시오.

## 요약 가이드라인

| 구분 | 전략 |
|----------|-----------|
| **가독성** | 복잡한 로직은 커스텀 훅으로 분리하고, 컴포넌트는 UI 렌더링에 집중하십시오. |
| **재사용성** | 작고 단일 책임(Single Responsibility)을 갖는 컴포넌트로 쪼개고 합성을 활용하십시오. |
| **성능** | 불필요한 리렌더링을 방지하기 위해 `memo`, `useMemo`를 적절히 사용하되 남용하지 마십시오. |
| **견고함** | 에러 바운더리(Error Boundary)를 설정하여 부분적인 에러가 전체 앱을 멈추지 않게 하십시오. |

**핵심 리마인더**: 현대적인 프론트엔드 패턴의 목표는 유지보수가 쉽고 성능이 뛰어난 UI를 구축하는 것입니다. 프로젝트의 규모와 복잡도에 가장 적합한 패턴을 선택하십시오.
