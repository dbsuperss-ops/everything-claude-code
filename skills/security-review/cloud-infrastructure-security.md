| 이름 | 설명 |
|------|-------------|
| cloud-infrastructure-security | 클라우드 플랫폼 배포, 인프라 구성, IAM 정책 관리, 로깅/모니터링 설정 또는 CI/CD 파이프라인 구현 시 이 스킬을 사용하십시오. 베스트 프랙티스에 맞춘 클라우드 보안 체크리스트를 제공합니다. |

# 클라우드 및 인프라 보안 스킬 (Cloud & Infrastructure Security Skill)

이 스킬은 클라우드 인프라, CI/CD 파이프라인 및 배포 구성이 보안 베스트 프랙티스를 따르고 업계 표준을 준수하도록 보장합니다.

## 활성화 시점

- 클라우드 플랫폼(AWS, Vercel, Railway, Cloudflare 등)에 애플리케이션을 배포할 때
- IAM 역할 및 권한을 구성할 때
- CI/CD 파이프라인을 설정할 때
- 코드형 인프라(IaC; Terraform, CloudFormation 등)를 구현할 때
- 로깅 및 모니터링을 구성할 때
- 클라우드 환경에서 비밀 정보(Secrets)를 관리할 때
- CDN 및 엣지(Edge) 보안을 설정할 때
- 재해 복구(DR) 및 백업 전략을 수립할 때

## 클라우드 보안 체크리스트

### 1. IAM 및 액세스 제어

#### 최소 권한 원칙 (Principle of Least Privilege)

```yaml
# ✅ 올바른 예: 최소한의 권한 부여
iam_role:
  permissions:
    - s3:GetObject  # 읽기 권한만 부여
    - s3:ListBucket
  resources:
    - arn:aws:s3:::my-bucket/*  # 특정 버킷으로 한정

# ❌ 잘못된 예: 지나치게 광범위한 권한 부여
iam_role:
  permissions:
    - s3:*  # 모든 S3 액션 허용
  resources:
    - "*"  # 모든 리소스 허용
```

#### 다요소 인증 (MFA)

```bash
# 루트/관리자 계정에는 반드시 MFA를 활성화하십시오.
aws iam enable-mfa-device \
  --user-name admin \
  --serial-number arn:aws:iam::123456789:mfa/admin \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

#### 검증 단계

- [ ] 프로덕션 환경에서 루트 계정을 사용하지 않음
- [ ] 모든 권한 있는 계정에 MFA가 활성화됨
- [ ] 서비스 계정은 영구적인 자격 증명이 아닌 역할(Roles)을 사용함
- [ ] IAM 정책이 최소 권한 원칙을 준수함
- [ ] 정기적인 액세스 리뷰를 수행함
- [ ] 사용되지 않는 자격 증명을 교체하거나 제거함

### 2. 비밀 정보 관리 (Secrets Management)

#### 클라우드 비밀 관리 도구 사용

```typescript
// ✅ 올바른 예: 클라우드 비밀 관리 도구 활용
import { SecretsManager } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManager({ region: 'us-east-1' });
const secret = await client.getSecretValue({ SecretId: 'prod/api-key' });
const apiKey = JSON.parse(secret.SecretString).key;

// ❌ 잘못된 예: 하드코딩하거나 환경 변수만 사용
const apiKey = process.env.API_KEY; // 교체되지 않으며 감사 추적이 불가능함
```

#### 비밀번호 교체 (Rotation)

```bash
# 데이터베이스 자격 증명에 대해 자동 교체 설정
aws secretsmanager rotate-secret \
  --secret-id prod/db-password \
  --rotation-lambda-arn arn:aws:lambda:region:account:function:rotate \
  --rotation-rules AutomaticallyAfterDays=30
```

#### 검증 단계

- [ ] 모든 비밀 정보를 클라우드 관리 도구(AWS Secrets Manager, Vercel Secrets 등)에 저장함
- [ ] 데이터베이스 자격 증명에 대해 자동 교체가 활성화됨
- [ ] API 키를 최소 분기별로 교체함
- [ ] 코드, 로그 또는 에러 메시지에 비밀 정보가 노출되지 않음
- [ ] 비밀 정보 액세스에 대해 감사 로깅(Audit logging)이 활성화됨

### 3. 네트워크 보안

#### VPC 및 방화벽 구성

```terraform
# ✅ 올바른 예: 제한된 보안 그룹 설정
resource "aws_security_group" "app" {
  name = "app-sg"
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # 내부 VPC만 허용
  }
  
  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # HTTPS 외부 전송만 허용
  }
}

# ❌ 잘못된 예: 전체 인터넷에 개방
resource "aws_security_group" "bad" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # 모든 포트, 모든 IP 허용!
  }
}
```

#### 검증 단계

- [ ] 데이터베이스가 공용 인터넷에서 접속 불가능함
- [ ] SSH/RDP 포트가 VPN/Bastion 호스트로만 제한됨
- [ ] 보안 그룹(Security Groups)이 최소 권한 원칙을 따름
- [ ] 네트워크 ACL이 올바르게 구성됨
- [ ] VPC 플로우 로그(Flow logs)가 활성화됨

### 4. 로깅 및 모니터링

#### CloudWatch/로깅 구성

```typescript
// ✅ 올바른 예: 종합적인 로깅 수행
import { CloudWatchLogsClient, CreateLogStreamCommand } from '@aws-sdk/client-cloudwatch-logs';

const logSecurityEvent = async (event: SecurityEvent) => {
  await cloudwatch.putLogEvents({
    logGroupName: '/aws/security/events',
    logStreamName: 'authentication',
    logEvents: [{
      timestamp: Date.now(),
      message: JSON.stringify({
        type: event.type,
        userId: event.userId,
        ip: event.ip,
        result: event.result,
        // 민감한 데이터는 절대 로깅하지 마십시오.
      })
    }]
  });
};
```

#### 검증 단계

- [ ] 모든 서비스에 대해 CloudWatch/로깅이 활성화됨
- [ ] 인증 실패 시도가 기록됨
- [ ] 관리자 액션이 감사됨
- [ ] 로그 보존 기간이 설정됨 (규정 준수를 위해 90일 이상 권장)
- [ ] 의심스러운 활동에 대해 알람이 구성됨
- [ ] 로그가 중앙화되고 변조 방지 처리됨

### 5. CI/CD 파이프라인 보안

#### 안전한 파이프라인 구성

```yaml
# ✅ 올바른 예: 안전한 GitHub Actions 워크플로우
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read  # 최소 권한 부여
      
    steps:
      - uses: actions/checkout@v4
      
      # 비밀 정보 유출 스캔
      - name: Secret scanning
        uses: trufflesecurity/trufflehog@main
        
      # 의존성 감사
      - name: Audit dependencies
        run: npm audit --audit-level=high
        
      # 장기 자격 증명 대신 OIDC 사용
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
          aws-region: us-east-1
```

#### 공급망 보안 (Supply Chain Security)

```json
// package.json - 락(Lock) 파일 및 무결성 검사 사용
{
  "scripts": {
    "install": "npm ci",  // 재현 가능한 빌드를 위해 ci 사용
    "audit": "npm audit --audit-level=moderate",
    "check": "npm outdated"
  }
}
```

#### 검증 단계

- [ ] 장기 자격 증명 대신 OIDC를 사용함
- [ ] 파이프라인 내에서 비밀 정보 스캔을 수행함
- [ ] 의존성 취약점 스캔을 수행함
- [ ] 컨테이너 이미지 스캔을 수행함 (해당하는 경우)
- [ ] 브랜치 보호 규칙이 적용됨
- [ ] 머지(Merge) 전 코드 리뷰가 필수임
- [ ] 서명된 커밋(Signed commits)만 허용함

### 6. Cloudflare 및 CDN 보안

#### Cloudflare 보안 구성

```typescript
// ✅ 올바른 예: 보안 헤더를 포함한 Cloudflare Workers
export default {
  async fetch(request: Request): Promise<Response> {
    const response = await fetch(request);
    
    // 보안 헤더 추가
    const headers = new Headers(response.headers);
    headers.set('X-Frame-Options', 'DENY');
    headers.set('X-Content-Type-Options', 'nosniff');
    headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
    headers.set('Permissions-Policy', 'geolocation=(), microphone=()');
    
    return new Response(response.body, {
      status: response.status,
      headers
    });
  }
};
```

#### WAF 규칙

```bash
# Cloudflare WAF 관리형 규칙 활성화
# - OWASP 코어 규칙셋
# - Cloudflare 관리형 규칙셋
# - 속도 제한(Rate limiting) 규칙
# - 봇(Bot) 보호
```

#### 검증 단계

- [ ] OWASP 규칙이 포함된 WAF가 활성화됨
- [ ] 속도 제한이 구성됨
- [ ] 봇 보호가 활성화됨
- [ ] DDoS 보호가 활성화됨
- [ ] 보안 헤더가 구성됨
- [ ] SSL/TLS Strict 모드가 활성화됨

### 7. 백업 및 재해 복구

#### 자동 백업 설정

```terraform
# ✅ 올바른 예: 자동 RDS 백업
resource "aws_db_instance" "main" {
  allocated_storage     = 20
  engine               = "postgres"
  
  backup_retention_period = 30  # 30일 보존
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"
  
  enabled_cloudwatch_logs_exports = ["postgresql"]
  
  deletion_protection = true  # 실수로 인한 삭제 방지
}
```

#### 검증 단계

- [ ] 매일 자동 백업이 수행되도록 구성됨
- [ ] 백업 보존 기간이 규정 준수 요구사항을 충족함
- [ ] 특정 시점 복구(PITR)가 활성화됨
- [ ] 백업 테스트를 분기별로 수행함
- [ ] 재해 복구 계획이 문서화됨
- [ ] RPO 및 RTO가 정의되고 테스트됨

## 배포 전 클라우드 보안 체크리스트

프로덕션 클라우드 배포를 수행하기 전 반드시 확인하십시오:

- [ ] **IAM**: 루트 계정 미사용, MFA 활성화, 최소 권한 정책 적용
- [ ] **Secrets**: 모든 비밀 정보를 자동 교체되는 클라우드 관리 도구에 저장
- [ ] **Network**: 보안 그룹 제한, 데이터베이스의 외부 노출 부재
- [ ] **Logging**: 보존 기간이 설정된 CloudWatch/로깅 활성화
- [ ] **Monitoring**: 이상 징후에 대한 알람 구성
- [ ] **CI/CD**: OIDC 인증, 비밀 정보 스캔, 의존성 감사 수행
- [ ] **CDN/WAF**: OWASP 규칙이 적용된 Cloudflare WAF 활성화
- [ ] **Encryption**: 저장 데이터(At rest) 및 전송 데이터(In transit) 암호화
- [ ] **Backups**: 복구 테스트가 완료된 자동 백업 구성
- [ ] **Compliance**: GDPR/HIPAA 등 관련 요구사항 충족 여부 (해당 시)
- [ ] **Documentation**: 인프라 문서화 및 업무 처리 절차서(Runbooks) 작성
- [ ] **Incident Response**: 보안 사고 대응 계획 수립

## 흔히 발생하는 클라우드 보안 설정 오류

### S3 버킷 노출

```bash
# ❌ 잘못된 예: 공개 버킷
aws s3api put-bucket-acl --bucket my-bucket --acl public-read

# ✅ 올바른 예: 특정 액세스만 허용하는 비공개 버킷
aws s3api put-bucket-acl --bucket my-bucket --acl private
aws s3api put-bucket-policy --bucket my-bucket --policy file://policy.json
```

### RDS 공용 액세스 허용

```terraform
# ❌ 잘못된 예
resource "aws_db_instance" "bad" {
  publicly_accessible = true  # 절대 금물!
}

# ✅ 올바른 예
resource "aws_db_instance" "good" {
  publicly_accessible = false
  vpc_security_group_ids = [aws_security_group.db.id]
}
```

## 참고 자료 (영어)

- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [Cloudflare Security Documentation](https://developers.cloudflare.com/security/)
- [OWASP Cloud Security](https://owasp.org/www-project-cloud-security/)
- [Terraform Security Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/)

**기억하십시오**: 클라우드 설정 오류는 데이터 유출의 주요 원인입니다. 단 하나의 노출된 S3 버킷이나 지나치게 허용적인 IAM 정책만으로도 전체 인프라가 위험에 처할 수 있습니다. 항상 최소 권한 원칙과 심층 방어 전략을 따르십시오.
