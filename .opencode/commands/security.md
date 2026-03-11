---
description: 종합 보안 리뷰 실행
agent: security-reviewer
subtask: true
---

# 보안 리뷰 명령 (Security Review Command)

종합적인 보안 리뷰를 수행합니다: $ARGUMENTS

## 임무

OWASP 가이드라인 및 보안 베스트 프랙티스에 따라 지정된 코드의 보안 취약점을 분석하십시오.

## 보안 체크리스트

### OWASP Top 10

1. **인젝션 (Injection)** (SQL, NoSQL, OS 명령, LDAP)
   - 매개변수화된 쿼리 사용 여부 확인
   - 입력값 정제(Sanitization) 검증
   - 동적 쿼리 생성 방식 검토

2. **취약한 인증 (Broken Authentication)**
   - 비밀번호 저장 방식 (bcrypt, argon2 등)
   - 세션 관리
   - 다요소 인증 (MFA)
   - 비밀번호 초기화 흐름

3. **민감한 데이터 노출 (Sensitive Data Exposure)**
   - 저장 시 및 전송 시 암호화
   - 적절한 키 관리
   - 개인 식별 정보 (PII) 처리

4. **XML 외부 개체 (XXE)**
   - DTD 프로세싱 비활성화 여부
   - XML에 대한 입력값 검증

5. **취약한 접근 제어 (Broken Access Control)**
   - 모든 엔드포인트에서의 권한 확인
   - 역할 기반 접근 제어 (RBAC)
   - 리소스 소유권 검증

6. **보안 설정 오류 (Security Misconfiguration)**
   - 기본 자격 증명 제거 여부
   - 에러 처리를 통한 정보 유출 방지
   - 보안 헤더 구성 확인

7. **크로스 사이트 스크립팅 (XSS)**
   - 출력 인코딩
   - 콘텐츠 보안 정책 (CSP)
   - 입력값 정제

8. **안전하지 않은 역직렬화 (Insecure Deserialization)**
   - 직렬화된 데이터의 유효성 검사
   - 무결성 검사 구현 여부

9. **알려진 취약점이 있는 구성 요소 사용**
   - `npm audit` 실행
   - 오래된 종속성 확인

10. **불충분한 로깅 및 모니터링**
    - 보안 이벤트 로그 기록 여부
    - 로그 내 민감한 데이터 포함 여부
    - 알림 설정 구성 확인

### 추가 점검 사항

- [ ] 코드 내 비밀 정보(API 키, 비밀번호) 포함 여부
- [ ] 환경 변수 처리 방식
- [ ] CORS 설정
- [ ] 속도 제한 (Rate limiting)
- [ ] CSRF 보호
- [ ] 보안 쿠키 플래그 (Secure cookie flags)

## 보고서 형식

### 치명적 이슈 (Critical Issues)
[즉시 수정해야 하는 문제들]

### 높은 우선순위 (High Priority)
[릴리스 전까지 수정해야 하는 문제들]

### 권장 사항 (Recommendations)
[고려해야 할 보안 개선 사항들]

---

**중요**: 보안 이슈는 진행 차단 요소(Blocker)입니다. 치명적인 이슈가 해결될 때까지 작업을 계속하지 마십시오.
