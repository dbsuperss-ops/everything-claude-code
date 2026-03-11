---
name: cloud-infrastructure-security
description: 클라우드 플랫폼 배포, 인프라 구성, IAM 정책 관리, 로깅/모니터링 설정 또는 CI/CD 파이프라인 구현 시 이 스킬을 사용하십시오. 베스트 프랙티스에 기반한 클라우드 보안 체크리스트를 제공합니다.
origin: ECC
---

# 클라우드 및 인프라 보안 스킬

이 스킬은 클라우드 인프라, CI/CD 파이프라인 및 배포 설정이 보안 베스트 프랙티스와 업계 표준을 준수하도록 보장합니다.

## 적용 시점

* 애플리케이션을 클라우드 플랫폼(AWS, Vercel, Railway, Cloudflare 등)에 배포할 때
* IAM 역할(Role) 및 권한을 설정할 때
* CI/CD 파이프라인을 구축할 때
* 코드형 인프라(IaC - Terraform, CloudFormation 등)를 구현할 때
* 로깅 및 모니터링 시스템을 구성할 때
* 클라우드 환경에서 비밀 정보(Secrets)를 관리할 때
* CDN 및 에지(Edge) 보안을 설정할 때
* 재해 복구(DR) 및 백업 전략을 수립할 때

## 클라우드 보안 체크리스트

### 1. IAM 및 접근 제어

#### 최소 권한 원칙 (Least Privilege)
```yaml
# ✅ 권장: 최소 권한 부여
iam_role:
  permissions:
    - s3:GetObject  # 읽기 전용 권한만 부여
    - s3:ListBucket
  resources:
    - arn:aws:s3:::my-bucket/*  # 특정 버킷으로만 범위 제한

# ❌ 위험: 과도한 권한 부여
iam_role:
  permissions:
    - s3:*  # 모든 S3 작업 허용
  resources:
    - "*"  # 모든 리소스에 접근 가능
```

#### 다요소 인증 (MFA)
* 루트(Root) 및 관리자(Admin) 계정에는 반드시 MFA를 활성화하십시오.

**검증 단계:**
* [ ] 운영 환경에서 루트 계정을 사용하지 않는가?
* [ ] 모든 권한 있는 계정에 MFA가 설정되었는가?
* [ ] 서비스 계정은 고정된 자격 증명 대신 IAM 역할을 사용하는가?
* [ ] IAM 정책이 최소 권한 원칙을 따르는가?
* [ ] 사용하지 않는 자격 증명은 정기적으로 삭제/교체되는가?

### 2. 비밀 정보 관리 (Secrets Management)

#### 클라우드 비밀 관리 도구 활용
```typescript
// ✅ 권장: 클라우드 전용 비밀 관리 서비스 사용 (예: AWS Secrets Manager)
import { SecretsManager } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManager({ region: 'us-east-1' });
const secret = await client.getSecretValue({ SecretId: 'prod/api-key' });
```

#### 자동 교체 (Rotation)
* 데이터베이스 비밀번호나 API 키는 30일~90일 주기로 자동 교체되도록 설정하십시오.

### 3. 네트워크 보안 (Network Security)

* **VPC 및 보안 그룹**: 데이터베이스는 공용 인터넷에 노출하지 마십시오. SSH/RDP 포트는 VPN 또는 배스천 호스트(Bastion host)를 통해서만 접근 가능하도록 제한하십시오.
* **보안 그룹 설정**: 화이트리스트 기반으로 필요한 포트(예: 443)와 IP 범위만 허용하십시오.

### 4. 로깅 및 모니터링

* 모든 서비스에 로깅을 활성화하십시오.
* 인증 실패 시도, 관리자 작업(Admin actions) 등의 이벤트는 반드시 로그로 기록하십시오.
* 로그 보존 기간은 규제 준수 요건(보통 90일 이상)에 맞춰 설정하십시오.
* 의심스러운 활동 발견 시 즉시 알림(Alerting)이 가도록 구성하십시오.

### 5. CI/CD 파이프라인 보안

* **OIDC 사용**: 정적이고 수명이 긴 토큰 대신 OIDC(OpenID Connect)를 사용하여 클라우드에 인증하십시오.
* **비밀 정보 스캔**: 파이프라인 단계에서 코드 내 비밀 정보(Secrets) 노출 여부를 자동 스캔하십시오.
* **의존성 감사**: 배포 전 `npm audit` 등으로 서드파티 라이브러리 취약점을 점검하십시오.

### 6. 에지(Edge) 및 CDN 보안 (Cloudflare 등)

* WAF(Web Application Firewall)를 활성화하고 OWASP 규칙을 적용하십시오.
* 속도 제한(Rate Limiting) 및 봇 차단 설정을 적용하십시오.
* 보안 헤더(X-Frame-Options, HSTS 등)를 추가하십시오.

### 7. 백업 및 재해 복구 (Backup & DR)

* 매일 자동 백업을 수행하고, 백업본의 보존 기간이 요건을 충족하는지 확인하십시오.
* 정기적으로(예: 매 분기) 백업 복구 테스트를 실시하십시오.
* RPO(복구 지점 목표)와 RTO(복구 시간 목표)가 정의되고 테스트되었는지 확인하십시오.

---

## 클라우드 배포 전 보안 체크리스트

* [ ] **IAM**: 루트 계정 미사용, MFA 활성화, 최소 권한 정책 적용 완료
* [ ] **비밀 정보**: 클라우드 비밀 관리자 사용 및 자동 교체 설정 완료
* [ ] **네트워크**: 보안 그룹 차단 완료, 공용 DB 노출 없음
* [ ] **로깅/모니터링**: 핵심 이벤트 로깅 및 이상 징후 알림 설정 완료
* [ ] **CI/CD**: OIDC 사용, 코드 내 비밀 정보 노출 스캔 통과
* [ ] **보안 헤더**: CSP, HSTS, X-Frame-Options 등 설정 완료
* [ ] **백업**: 자동 백업 및 복구 테스트 완료

**핵심**: 클라우드 설정 오류는 데이터 유출의 가장 큰 원인입니다. 노출된 S3 버킷이나 과도한 권한의 IAM 정책 하나가 전체 인프라를 위험에 빠뜨릴 수 있습니다. 항상 최소 권한 원칙과 다층 방어(Defense in Depth) 전략을 고수하십시오.
