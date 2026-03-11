---
name: springboot-security
description: Java Spring Boot 서비스의 인증/인가, 유효성 검증, CSRF, 비밀 정보 관리, 보안 헤더, 속도 제한 및 의존성 보안을 위한 Spring Security 베스트 프랙티스입니다.
origin: ECC
---

# Spring Boot 보안 리뷰

인증 추가, 입력 처리, 엔드포인트 생성 또는 비밀 정보를 다룰 때 이 스킬을 활용하십시오.

## 적용 시점

* 인증 방식(JWT, OAuth2, 세션 기반)을 추가할 때
* 권한 부여(@PreAuthorize, 역할 기반 접근 제어)를 구현할 때
* 사용자 입력 검증(Bean Validation, 커스텀 검증기)을 수행할 때
* CORS, CSRF 또는 보안 헤더를 설정할 때
* 비밀 정보(Vault, 환경 변수)를 관리할 때
* 속도 제한(Rate Limiting) 또는 무차별 대입 공격 방어 로직을 추가할 때
* 취약점(CVE) 점검을 위해 의존성을 스캔할 때

## 인증 (Authentication)

* **무상태성(Stateless)**: JWT 또는 취소 리스트가 포함된 불투명 토큰(Opaque token) 사용을 우선하십시오.
* **쿠키 설정**: 세션 사용 시 `httpOnly`, `Secure`, `SameSite=Strict` 옵션을 반드시 적용하십시오.
* **필터 활용**: `OncePerRequestFilter`를 상속받아 토큰 검증 로직을 구현하십시오.

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) {
    String header = request.getHeader(HttpHeaders.AUTHORIZATION);
    if (header != null && header.startsWith("Bearer ")) {
      // 토큰 검증 및 SecurityContext 설정 로직
    }
  }
}
```

## 인가 (Authorization)

* **메소드 보안 활성화**: `@EnableMethodSecurity`를 사용하십시오.
* **권한 체크**: `@PreAuthorize("hasRole('ADMIN')")` 또는 커스텀 빈을 활용한 `@PreAuthorize("@authz.isOwner(#id, authentication)")`를 사용하십시오.
* **기본 거부 원칙**: 모든 접근을 기본적으로 차단하고 필요한 범위만 명시적으로 허용하십시오.

## 입력값 검증 (Validation)

* **Bean Validation**: 컨트롤러에서 `@Valid`와 함께 DTO 수준의 제약 조건(`@NotBlank`, `@Email`, `@Size` 등)을 적용하십시오.
* **DTO 사용**: API 입력에 엔티티를 직접 사용하지 말고 전용 DTO를 활용하십시오.

## SQL 인젝션 방지

* **Spring Data JPA**: 가급적 쿼리 메소드를 사용하고, 네이티브 쿼리 사용 시 반드시 파라미터 바인딩(`:name`)을 사용하십시오. 절대 문자열을 직접 결합하지 마십시오.

## 비밀번호 관리

* **암호화**: `BCryptPasswordEncoder` 또는 `Argon2`를 사용하여 해싱하십시오. 평문 저장은 절대 금지입니다.

## CSRF 및 CORS

* **CSRF**: 브라우저 기반 세션 앱은 CSRF를 활성화하고, 순수 API 기반(JWT 사용)은 비활성화하되 무상태 인증에 의존하십시오.
* **CORS**: 컨트롤러 단위가 아닌 보안 필터 수준에서 통합 관리하고, 운영 환경에서 허용 도메인(`AllowedOrigins`)에 와일드카드(`*`)를 사용하지 마십시오.

## 비밀 정보 및 로그 관리

* **환경 변수**: 소스 코드나 `application.yml`에 비밀번호를 기록하지 말고 `${DB_PASSWORD}`와 같은 환경 변수 플레이스홀더를 사용하십시오.
* **민감 정보 마스킹**: 로그에 비밀번호, 토큰, 신용카드 번호 등이 남지 않도록 주의하십시오.

---

## 배포 전 최종 체크리스트

* [ ] 토큰 검증 및 만료 처리가 정확한가?
* [ ] 모든 민감한 경로에 권한 체크 로직이 있는가?
* [ ] 입력값에 대한 검증 및 새니타이징이 수행되는가?
* [ ] SQL 인젝션 위험(문자열 결합 쿼리)이 없는가?
* [ ] 비밀 정보가 환경 변수로 외부화되었는가?
* [ ] 보안 헤더(CSP, HSTS 등)가 설정되었는가?
* [ ] 주요 엔드포인트에 속도 제한이 적용되었는가?
* [ ] 의존성 라이브러리에 알려진 취약점이 없는가?

**핵심**: 보안의 기본은 '기본 거부(Default Deny)', '입력값 불신', '최소 권한 부여'입니다. 안전한 설정을 우선적으로 적용하십시오.
