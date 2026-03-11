---
description: "공통 규칙을 확장하는 PHP 보안"
globs: ["**/*.php", "**/composer.lock", "**/composer.json"]
alwaysApply: false
---
# PHP 보안 (Security)

> 이 문서는 공통 보안 규칙을 기반으로 PHP에 특화된 내용을 확장합니다.

## 데이터베이스 안전성

- 모든 동적 쿼리에 대해 준비된 문구(Prepared statements, `PDO`, Doctrine, Eloquent 쿼리 빌더 등)를 사용하십시오.
- ORM의 대량 할당(Mass-assignment) 범위를 신중하게 설정하고 쓰기 가능한 필드를 허용 목록(Whitelist)으로 관리하십시오.

## 비밀 정보 및 의존성

- 비밀 정보는 커밋된 구성 파일이 아니라 환경 변수나 비밀 정보 관리 도구(Secret Manager)에서 로드하십시오.
- CI에서 `composer audit`을 실행하고, 의존성을 추가하기 전에 패키지 신뢰성을 검토하십시오.

## 인증 및 세션 안전성

- 비밀번호 저장에는 `password_hash()` / `password_verify()`를 사용하십시오.
- 인증 및 권한 변경 후에는 세션 식별자를 재생성하십시오.
- 상태를 변경하는 웹 요청에 대해 CSRF 보호 기능을 강제하십시오.
