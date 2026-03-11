---
description: 최소한의 변경으로 빌드 및 TypeScript 에러 수정
agent: build-error-resolver
subtask: true
---

# 빌드 수정 명령 (Build Fix Command)

최소한의 변경으로 빌드 및 TypeScript 에러를 수정합니다: $ARGUMENTS

## 임무

1. **타입 체크 실행**: `npx tsc --noEmit`
2. **모든 에러 수집**
3. **에러를 하나씩 수정** (최소한의 변경으로)
4. **각 수정 사항이 새로운 에러를 유발하지 않는지 확인**
5. **모든 에러가 해결되었는지 최종 확인**

## 접근 방식

### 권장 사항 (DO):
- ✅ 올바른 타입을 사용하여 타입 에러 수정
- ✅ 누락된 임포트(Import) 추가
- ✅ 구문(Syntax) 에러 수정
- ✅ 최소한의 변경 유지
- ✅ 기존 동작 보존
- ✅ 각 변경 후 `tsc --noEmit` 실행

### 금지 사항 (DON'T):
- ❌ 코드 리팩토링
- ❌ 새로운 기능 추가
- ❌ 아키텍처 변경
- ❌ `any` 타입 사용 (절대적으로 필요한 경우 제외)
- ❌ `@ts-ignore` 주석 추가
- ❌ 비즈니스 로직 변경

## 일반적인 에러 수정

| 에러 | 해결 방법 |
|-------|-----|
| Type 'X' is not assignable to type 'Y' | 올바른 타입 어노테이션 추가 |
| Property 'X' does not exist | 인터페이스에 속성 추가 또는 속성 이름 수정 |
| Cannot find module 'X' | 패키지 설치 또는 임포트 경로 수정 |
| Argument of type 'X' is not assignable | 캐스팅 또는 함수 서명 수정 |
| Object is possibly 'undefined' | null 체크 또는 옵셔널 체이닝 추가 |

## 검증 단계

수정 후:
1. `npx tsc --noEmit` - 에러 0개여야 함
2. `npm run build` - 성공해야 함
3. `npm test` - 테스트가 여전히 통과해야 함

---

**중요**: 오직 에러를 수정하는 데만 집중하십시오. 리팩토링, 개선, 아키텍처 변경은 수행하지 마십시오. 최소한의 차이(diff)로 빌드를 성공(Green) 상태로 만드십시오.
