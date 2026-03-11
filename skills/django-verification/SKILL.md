---
name: django-verification
description: "Django 프로젝트를 위한 검증 루프: 마이그레이션, 린팅, 테스트 커버리지, 보안 스캔 및 배포 전 최종 점검 가이드입니다."
origin: ECC
---

# Django 검증 루프 (Django Verification Loop)

PR(Pull Request) 생성 전, 주요 변경 사항 적용 후, 그리고 최종 배포 전에 실행하여 Django 애플리케이션의 품질과 보안을 보장합니다.

## 활성화 시점

- Django 프로젝트의 PR을 열기 전
- 모델 변경, 마이그레이션 업데이트 또는 종속성 업그레이드 후
- 스테이징 또는 상용 환경 배포 전 검증 시
- 환경 설정 -> 린트 -> 테스트 -> 보안 -> 배포 준비로 이어지는 전체 파이프라인 실행 시
- 마이그레이션의 안전성 및 테스트 커버리지 확인 시

## 단계 1: 환경 점검 (Environment Check)

Python 버전, 가상 환경 상태 및 필수 환경 변수(`DJANGO_SECRET_KEY` 등)가 제대로 설정되었는지 확인합니다.

## 단계 2: 코드 품질 및 포매팅 (Code Quality & Formatting)

- **Mypy**: 타입 검사 수행
- **Ruff**: 린팅 수행 및 자동 수정
- **Black**: 코드 포매팅 확인 및 자동 수정
- **Isort**: 임포트(Import) 정렬 확인 및 자동 수정
- **Django Check**: `python manage.py check --deploy`로 배포용 설정 확인

## 단계 3: 마이그레이션 (Migrations)

- 적용되지 않은 마이그레이션이 있는지 확인합니다.
- 마이그레이션 충돌(`Conflicts`) 여부를 점검합니다.
- 모델 변경 사항이 마이그레이션 파일에 모두 반영되었는지 확인합니다.

## 단계 4: 테스트 및 커버리지 (Tests + Coverage)

`pytest`를 사용하여 모든 테스트를 실행하고 커버리지 보고서를 생성합니다.
- **목표 커버리지**: 전체 80% 이상, 핵심 비즈니스 로직(모델/서비스) 90% 이상 권장.

## 단계 5: 보안 스캔 (Security Scan)

- **pip-audit / safety**: 종속 라이브러리의 보안 취약점 점검
- **Bandit**: 파이썬 코드 보안 린팅
- **Gitleaks**: 하드코딩된 비밀 키/자격 증명 감지
- **DEBUG 모드 확인**: 상용 환경에서는 반드시 `False`여야 함.

## 단계 6: Django 관리 명령 (Management Commands)

- `collectstatic`: 정적 파일 모으기 기능 확인
- `check --database`: 데이터베이스 무결성 확인
- 캐시 백엔드(Redis 등) 연결 테스트

## 단계 7: 성능 점검 (Performance Checks)

- **N+1 쿼리**: SQL 패널을 통해 중복 쿼리 여부 확인
- **인덱스**: 자주 사용되는 쿼리 경로에 인덱스가 누락되었는지 확인
- **쿼리 횟수**: 페이지당 쿼리 횟수가 적절한 수준(보통 50회 미만)인지 확인

## 단계 8: 정적 자산 점검 (Static Assets)

npm 종속성 보안 점검 및 프론트엔드 빌드 결과물을 확인합니다.

## 단계 9: 설정 리뷰 (Configuration Review)

`DEBUG`, `SECRET_KEY`, `ALLOWED_HOSTS`, HTTPS/HSTS 설정 등 배포 필수 항목들이 올바르게 구성되었는지 최종 확인합니다.

## 단계 10: 로깅 및 모니터링 (Logging)

로그 파일 기록 권한 및 에러 모니터링 도구(Sentry 등)의 연동 상태를 확인합니다.

## 단계 11: API 문서화 (API Documentation)

DRF 사용 시 OpenAPI 스키마가 정상적으로 생성되는지, Swagger UI 접근이 가능한지 확인합니다.

## 단계 12: Diff 리뷰 (Diff Review)

`git diff`를 통해 실제 변경 로그를 검토하고, 디버깅 코드(`print`, `pdb`)나 `TODO` 주석이 남아있는지 확인합니다.

## 검증 보고서 템플릿 (Output Template)

검증 완료 후에는 각 단계별 성공(✓) 및 실패(✗) 여부와 조치 필요 사항(권장 사항)을 포함한 보고서를 작성하십시오.

**기억하십시오**: 자동화된 검증은 공통적인 실수를 잡아내기에 매우 유용하지만, 사람이 직접 수행하는 코드 리뷰와 스테이징 환경에서의 최종 테스트를 완전히 대신할 수는 없습니다.
    
