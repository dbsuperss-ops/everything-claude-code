---
description: Next.js 15, Supabase 및 Stripe를 사용하는 SaaS 애플리케이션 프로젝트의 CLAUDE.md 설정 예시입니다.
---

# SaaS 애플리케이션 프로젝트 가이드 (CLAUDE.md 예시)

이 파일은 Next.js 기반의 SaaS 프로젝트에서 Claude Code가 준수해야 할 핵심 지침을 담고 있습니다.

## 프로젝트 개요

**기술 스택:** Next.js 15 (App Router), TypeScript, Supabase (Auth + DB), Stripe (결제), Tailwind CSS, Playwright (E2E 테스트).

**아키텍처:** 서버 컴포넌트 우선 원칙을 따릅니다. 상호작용이 필요한 경우에만 제한적으로 클라이언트 컴포넌트를 사용하십시오. 데이터 변경은 서버 액션(Server Actions)을, 외부 연동은 API 라우트(Webhooks)를 활용합니다.

## 핵심 규칙

### 1. 데이터베이스 및 보안 (Supabase)
* **RLS 준수**: 모든 쿼리는 행 단위 보안(RLS)이 활성화된 Supabase 클라이언트를 사용하며, RLS 우회는 금지됩니다.
* **마이그레이션**: `supabase/migrations/` 폴더의 스크립트로 스키마를 관리하고 직접 수정하지 마십시오.
* **최적화**: `select('*')` 대신 필요한 컬럼만 명시적으로 선택하고, 대량 데이터 방지를 위해 반드시 `.limit()`을 포함하십시오.

### 2. 인증 및 미들웨어
* **클라이언트 분리**: 서버에서는 `createServerClient()`, 클라이언트에서는 `createBrowserClient()`를 사용하십시오.
* **유저 검증**: 보호된 경로에서는 `getSession()`이 아닌 `getUser()`를 호출하여 위조된 세션이 아닌지 확인하십시오.
* **미들웨어**: `middleware.ts`를 통해 매 요청마다 인증 토큰을 갱신하십시오.

### 3. 결제 시스템 (Stripe)
* **신뢰의 기반**: 클라이언트에서 전달된 가격 정보는 절대 믿지 마십시오. 항상 서버에서 Stripe API를 통해 직접 확인하십시오.
* **상태 동기화**: `subscription_status` 컬럼은 Stripe Webhook(`app/api/webhooks/stripe/route.ts`)을 통해서만 업데이트하십시오.

### 4. 코드 스타일 및 패턴
* **컴포넌트 설계**: `'use client'` 지시어는 파일 최상단에 명시하고, 클라이언트 컴포넌트는 최소한의 로직(Hook 추출 등)만 포함하도록 가볍게 유지하십시오.
* **입력 검증**: 유효성 검사, API 요청, 환경 변수 파싱 등 모든 데이터 검증에는 Zod 스키마를 사용하십시오.

## 권장 파일 구조

```text
src/
  app/
    (auth)/          # 인증 관련 페이지 (로그인, 회원가입 등)
    (dashboard)/     # 보호된 대시보드 페이지
    api/webhooks/    # Stripe, Supabase 웹훅 핸들러
  components/ui/     # Shadcn/ui 등 재사용 컴포넌트
  lib/
    supabase/        # Supabase 클라이언트 팩토리
    stripe/          # Stripe 클라이언트 및 유틸리티
supabase/
  migrations/        # DB 마이그레이션 스크립트
```

## 주요 패턴 예시

### 서버 액션 패턴 (Server Action with Zod)
```typescript
'use server'

const schema = z.object({ name: z.string().min(1) })

export async function createProject(formData: FormData) {
  // 데이터 파싱, 유저 인증 확인, DB 삽입 로직
  // 성공 시 { success: true, data }, 실패 시 { success: false, error } 반환
}
```

## 주요 명령어 (Slash Commands)

* `/verify`: 빌드, 타입 체크, 린트 및 전체 보안 점검
* `/tdd`: 유닛 및 통합 테스트를 통한 기능 개발
* `/e2e`: Playwright를 활용한 핵심 사용자 여정(결제, 가입 등) 테스트
* `/plan`: 팀 초대 시스템이나 결제 연동 등 신규 기능 설계

**핵심**: 사용자 데이터 보안을 위해 RLS를 엄격히 관리하고, 서버와 클라이언트의 경계를 명확히 구분하여 성능과 보안을 동시에 잡으십시오.
