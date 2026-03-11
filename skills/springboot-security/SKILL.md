---
name: springboot-security
description: Java Spring Boot 서비스에서의 인증/인가, 유효성 검사, CSRF, 시크릿 관리, 보안 헤더, 속도 제한 및 의존성 보안을 위한 Spring Security 최선 관행 가이드입니다.
origin: ECC
---

# Spring Boot 보안 검토 (Spring Boot Security Review)

인증 기능을 추가하거나, 사용자 입력을 처리하고, 엔드포인트를 생성하거나, 시크릿(Secrets)을 다룰 때 이 스킬을 활용하십시오.

## 활성화 시점

- 인증(JWT, OAuth2, 세션 기반 등)을 추가할 때
- 인가 로직(`@PreAuthorize`, 역할 기반 접근 제어 등)을 구현할 때
- 사용자 입력값(Bean Validation, 커스텀 검증기 등)을 검증할 때
- CORS, CSRF, 또는 보안 헤더를 설정할 때
- 시크릿 관리(Vault, 환경 변수 등) 기능을 구현할 때
- 속도 제한(Rate Limiting) 또는 브루트 포스 방지 기능을 추가할 때
- 의존성 라이브러리의 취약점(CVE)을 스캔할 때

## 인증 및 인가 (Authn & Authz)

- **인증**: 상태 비저장(Stateless) JWT 또는 불투명 토큰(Opaque tokens) 사용을 권장합니다. 세션 사용 시 쿠키에 `httpOnly`, `Secure`, `SameSite=Strict` 속성을 적용하십시오.
- **인가**: `@EnableMethodSecurity`를 활성화하고, `@PreAuthorize`를 사용하여 메서드 레벨에서 접근을 제어하십시오. 기본적으로 모든 접근을 거부하고 필요한 범위만 허용하는 방식을 취하십시오.

## 입력값 검증 (Input Validation)

- 컨트롤러에서 `@Valid`와 Bean Validation을 사용하여 데이터를 검증하십시오.
- DTO에 `@NotBlank`, `@Email`, `@Size` 등의 제약 조건을 적용하십시오. 사용자로부터 받은 HTML은 렌더링 전 반드시 새니타이징(Sanitize)하십시오.

## 보안 강화 패턴

- **SQL 인젝션 방지**: 문자열 결합을 금지하고, 항상 매개변수화된 쿼리나 Spring Data 리포지토리를 사용하십시오.
- **비밀번호 암호화**: BCrypt나 Argon2를 사용하여 해싱하십시오. 절대 평문으로 저장하지 마십시오.
- **CSRF**: 브라우저 기반 세션 앱은 활성화하고, 순수 API(JWT 사용 시)는 비활성화하여 상태 비저장 인증에 의존하십시오.
- **시크릿 관리**: 소스 코드에는 시크릿을 남기지 말고 환경 변수나 Vault에서 로드시킵니다. `application.yml`에는 `${DB_PASSWORD}`와 같은 플레이스홀더를 사용하십시오.
- **보안 헤더 및 CORS**: CSP, Frame Options 등을 설정하고, CORS는 필터 레벨에서 신뢰할 수 있는 Origin(절대 `*` 금지)만 허용하도록 구성하십시오.

## 속도 제한 및 운영 보안

- Bucket4j 등을 사용하여 비용이 큰 엔드포인트에 속도 제한을 적용하십시오.
- CI 파이프라인에서 OWASP Dependency Check 등을 실행하여 의존성 취약점을 점검하십시오.
- 로그에 비밀번호, 토큰 등 민감한 정보를 남기지 않도록 주의하십시오.

**기억하십시오**: 기본적으로 거부(Deny by default)하고, 입력값을 철저히 검증하며, 최소 권한 원칙을 준수하십시오. 설정만으로 보안을 강화할 수 있는 부분을 먼저 고려하십시오.
    
