---
paths:
  - "**/*.php"
  - "**/composer.lock"
  - "**/composer.json"
---
# PHP 보안 (Security)

> 이 문서는 [common/security.md](../common/security.md)의 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## 입력 및 출력

- 프레임워크 경계에서 요청 입력을 검증하십시오 (`FormRequest`, Symfony Validator 또는 명시적인 DTO 검증 활용).
- 템플릿의 출력값은 기본적으로 이스케이프(Escape) 처리를 수행하십시오. Raw HTML 렌더링은 정당한 이유가 있는 예외적인 상황으로 간주해야 합니다.
- 쿼리 파라미터, 쿠키, 헤더 또는 업로드된 파일 메타데이터를 검증 없이 절대 신뢰하지 마십시오.

## 데이터베이스 안전성

- 모든 동적 쿼리에 대해 준비된 구문(Prepared statements; `PDO`, Doctrine, Eloquent 쿼리 빌더 등)을 사용하십시오.
- 컨트롤러나 뷰에서 문자열 결합 방식으로 SQL을 생성하는 것을 삼가십시오.
- ORM의 대량 할당(Mass-assignment) 범위를 신중하게 제한하고, 쓰기 가능한 필드를 화이트리스트(Whitelist)로 관리하십시오.

## 비밀 정보 및 종속성

- 비밀 정보는 환경 변수나 비밀 관리자(Secret manager)를 통해 로드하고, 커밋되는 설정 파일에 절대 포함하지 마십시오.
- CI 과정에서 `composer audit`을 실행하고, 새로운 패키지를 추가하기 전에 해당 패키지 관리자의 신뢰성을 검토하십시오.
- 주 버전(Major version)을 신중하게 고정(Pin)하고, 더 이상 관리되지 않는(Abandoned) 패키지는 조속히 제거하십시오.

## 인증 및 세션 안전성

- 비밀번호 저장 시 `password_hash()` / `password_verify()`를 사용하십시오.
- 인증 성공 후 또는 권한 변경 시 세션 식별자를 재생성하십시오.
- 상태를 변경하는 웹 요청에 대해 CSRF 보호를 강제 적용하십시오.
