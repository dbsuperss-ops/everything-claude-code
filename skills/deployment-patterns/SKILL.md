---
name: deployment-patterns
description: 배포 워크플로우, CI/CD 파이프라인 패턴, Docker 컨테이너화, 헬스 체크(Health checks), 롤백 전략 및 웹 애플리케이션의 운영 환경 배포 전 체크리스트 가이드입니다.
origin: ECC
---

# 배포 패턴 (Deployment Patterns)

운영 환경 배포 워크플로우 및 CI/CD 최선 관행(Best practices)을 안내합니다.

## 활성화 시점

- CI/CD 파이프라인 설정 시
- 애플리케이션을 Docker 컨테이너화할 때
- 배포 전략(Rolling, Blue-Green, Canary)을 수립할 때
- 헬스 체크(Health checks) 및 래디니스 프로브(Readiness probes) 구현 시
- 운영 서버 릴리스를 준비할 때
- 환경별(Environment-specific) 설정 구성 시

## 배포 전략 (Deployment Strategies)

### 롤링 배포 (Rolling Deployment) - 기본값
인스턴스를 점진적으로 교체합니다. 배포 중에는 구버전과 신버전이 동시에 실행됩니다.
- **장점**: 무중단 배포, 점진적 업데이트
- **단점**: 구버전/신버전 공존 기간에 하위 호환성 유지가 필수적임

### 블루-그린 배포 (Blue-Green Deployment)
두 개의 동일한 환경을 구성합니다. 트래픽을 한 번에 원자적으로 전환합니다.
- **장점**: 즉각적인 롤백 가능 (다시 블루로 전환), 깨끗한 절체(Cutover)
- **단점**: 배포 중 2배의 인프라 자원이 필요함

### 카나리 배포 (Canary Deployment)
소량의 트래픽을 신버전에 먼저 흘려보내 지표를 확인한 후 전체로 확대합니다.
- **장점**: 전체 배포 전 실제 트래픽으로 문제 조기 발견 가능
- **단점**: 트래픽 분할 인프라 및 모니터링 시스템 필요

## Docker 최선 관행

- **멀티 스테이지 빌드(Multi-stage build)**: 이미지 크기를 최소화하고 보안을 강화합니다.
- **사용자 권한**: root 유저가 아닌 `appuser`와 같은 일반 유저로 실행하십시오.
- **태그 사용**: `latest` 보다는 특정 버전 태그(예: `node:22-alpine`)를 사용하십시오.
- **헬스 체크**: `HEALTHCHECK` 명령어를 Dockerfile에 포함하여 컨테이너 상태를 감시하십시오.

## CI/CD 파이프라인 (GitHub Actions 예시)

1. **테스트 단계**: 린트, 타입 체크, 단위 테스트 및 통합 테스트 수행
2. **빌드 단계**: 테스트 통과 시 Docker 이미지 빌드 및 레지스트리 푸시
3. **배포 단계**: 환경별(Staging/Production) 배포 수행

## 헬스 체크 (Health Checks)

- **단순 체크**: 애플리케이션 생존 여부 확인
- **상세 체크**: 데이터베이스, Redis, 외부 API 등 의존 서비스들의 연결 상태까지 확인
- **Kubernetes 프로브**: Liveness(생존), Readiness(준비), Startup(시작) 프로브를 용도에 맞게 설정하십시오.

## 환경 설정 (Twelve-Factor App)

- 모든 설정은 소스 코드가 아닌 **환경 변수**를 통해 주입받아야 합니다.
- 애플리케이션 시작 시 `Zod`와 같은 라이브러리로 환경 변수의 유효성을 즉시 검증(Fail-Fast)하십시오.

## 롤백 전략 (Rollback Strategy)

- 쿠버네티스의 경우 `kubectl rollout undo` 등을 사용하여 이전 이미지로 즉시 되돌립니다.
- 배포 전 롤백 계획을 문서화하고 스테이징 환경에서 반드시 테스트해 보십시오.

## 운영 환경 배포 전 체크리스트 (Production Readiness)

- [ ] 모든 테스트 통과 및 보안 취약점 스캔 완료
- [ ] 코드나 설정에 하드코딩된 비밀 키가 없는가?
- [ ] 정형화된(JSON) 로깅 설정 및 개인정보(PII) 제거 확인
- [ ] 리소스 제한(CPU, Memory) 및 오토스케일링 설정
- [ ] 모든 엔드포인트에 SSL/TLS 적용
- [ ] 에러율 임계치 기반 알림(Alerting) 설정

**기억하십시오**: 안정적인 배포는 단순히 코드를 서버에 올리는 것이 아니라, 장애 상황에서 빠르게 복구할 수 있는 체계를 갖추는 것입니다.
    
