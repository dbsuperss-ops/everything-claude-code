---
name: django-verification
description: "Django 프로젝트를 위한 검증 루틴입니다. 마이그레이션 확인, 코드 린팅, 커버리지를 포함한 테스트 실행, 보안 스캔 및 배포 전 최종 체크리스트를 포함합니다."
origin: ECC
---

# Django 검증 루프 (Verification Loop)

PR(Pull Request) 생성 전, 주요 변경 사항 적용 후, 그리고 최종 배포 전에 실행하여 Django 애플리케이션의 품질과 보안을 보장하는 프로세스입니다.

## 적용 시점

* Django 프로젝트에서 PR을 올리기 직전
* 주요 모델 변경, 마이그레이션 업데이트 또는 의존성 라이브러리 업그레이드 후
* 스테이징(Staging) 및 운영(Production) 환경 배포 전 검증 단계
* 환경 설정 → 코드 품질 → 테스트 → 보안 → 배포 준비 상태를 종합적으로 점검할 때
* 마이그레이션의 안전성 및 테스트 커버리지 목표 달성 여부를 확인할 때

## 1단계: 환경 점검

```bash
# Python 버전 확인
python --version  # 프로젝트 요구 사양과 일치하는지 확인

# 가상 환경 상태 확인
which python
pip list --outdated # 업데이트가 필요한 패키지 확인

# 핵심 환경 변수 설정 확인
python -c "import os; print('DJANGO_SECRET_KEY 설정됨' if os.environ.get('DJANGO_SECRET_KEY') else '누락됨: DJANGO_SECRET_KEY')"
```

## 2단계: 코드 품질 및 포맷팅

```bash
# 타입 체크 (mypy)
mypy . --config-file pyproject.toml

# 린트 체크 및 자동 수정 (ruff)
ruff check . --fix

# 코드 포맷팅 체크 (black)
black . --check
black .  # 실제 자동 수정 적용 시

# 임포트 순서 정리 (isort)
isort . --check-only
isort .  # 실제 자동 수정 적용 시

# Django 배포용 설정 체크
python manage.py check --deploy
```

**주요 확인 사항:**
* 공용 함수에 타입 힌트가 누락되었는가
* PEP 8 스타일 가이드를 위반했는가
* 임포트(Import) 구문이 정렬되지 않았는가
* 운영 환경 설정에 개발용 DEBUG 설정이 남아있는가

## 3단계: 데이터베이스 마이그레이션

```bash
# 미적용 마이그레이션 확인
python manage.py showmigrations

# 누락된 마이그레이션 파일 확인 (모델 변경 후 생성 여부)
python manage.py makemigrations --check

# 마이그레이션 실행 계획 미리보기 (Dry-run)
python manage.py migrate --plan

# 마이그레이션 적용 (테스트 환경)
python manage.py migrate

# 마이그레이션 충돌 확인 및 병합
python manage.py makemigrations --merge
```

## 4단계: 테스트 및 커버리지

```bash
# pytest를 사용한 전체 테스트 및 커버리지 측정
pytest --cov=apps --cov-report=html --cov-report=term-missing --reuse-db

# 특정 앱만 테스트
pytest apps/users/tests/

# 마커(Marker)를 사용한 선별 실행
pytest -m "not slow"  # 느린 테스트 제외
pytest -m integration  # 통합 테스트만 실행

# 커버리지 보고서 확인
# (Windows의 경우) start htmlcov/index.html
```

**커버리지 목표치:**
| 컴포넌트 | 목표 수치 |
|-----------|--------|
| 모델 (Models) | 90% 이상 |
| 시리얼라이저 (Serializers) | 85% 이상 |
| 뷰 (Views) | 80% 이상 |
| 서비스 (Services) | 90% 이상 |
| **전체 평균** | **80% 이상** |

## 5단계: 보안 스캔

```bash
# 의존성 패키지 취약점 검사
pip-audit
safety check --full-report

# Django 보안 전용 체크
python manage.py check --deploy

# Bandit 보안 라이브러리 정적 분석
bandit -r . -f json -o bandit-report.json

# 비밀 정보(Secret) 유출 검사 (gitleaks 등 활용)
gitleaks detect --source . --verbose

# 운영 환경의 DEBUG 모드 설정 재확인
python -c "from django.conf import settings; print(f'DEBUG STATUS: {settings.DEBUG}')"
```

## 6단계: 핵심 관리 명령 실행

```bash
# 모델 구조 오류 체크
python manage.py check

# 정적 파일 수집 테스트
python manage.py collectstatic --noinput --clear

# 데이터베이스 연결성 체크
python manage.py check --database default

# 캐시(Redis 등) 연결 테스트
python -c "from django.core.cache import cache; cache.set('test', 'value', 10); print(f'Cache test: {cache.get('test')}')"
```

## 7단계: 성능 분석

* **N+1 쿼리 확인**: Django Debug Toolbar를 통해 SQL 패널에서 중복 쿼리 유무를 확인하십시오.
* **인덱스 누락 확인**: 자주 조회되는 필드에 데이터베이스 인덱스가 설정되어 있는지 검토하십시오.
* **페이지당 쿼리 수**: 일반적인 페이지 로드 시 쿼리 수가 50회 미만인지 확인하십시오.

## 8단계: 정적 자산(Static Assets) 검증

```bash
# NPM 취약점 검사 (프론트엔드 도구 사용 시)
npm audit

# 정적 파일 빌드 (Webpack, Vite 등)
npm run build

# 수집된 정적 파일 확인
ls -la staticfiles/
python manage.py findstatic css/style.css
```

## 9단계: 최종 설정(Config) 리뷰

```python
# Django 쉘에서 핵심 설정 최종 확인
python manage.py shell << EOF
from django.conf import settings
print(f"DEBUG is False: {not settings.DEBUG}")
print(f"SECRET_KEY 보안성: {len(settings.SECRET_KEY) > 30}")
print(f"ALLOWED_HOSTS 설정됨: {len(settings.ALLOWED_HOSTS) > 0}")
print(f"HTTPS 리다이렉트: {getattr(settings, 'SECURE_SSL_REDIRECT', False)}")
print(f"DB 엔진: {settings.DATABASES['default']['ENGINE']}")
EOF
```

## 10단계: Diff(변경사항) 리뷰

```bash
# 변경 파일 통계 확인
git diff --stat

# 실제 코드 차이점 상세 검토
git diff

# 흔한 실수(Debug 코드 등) 검색
git diff | grep -i "todo\|fixme\|hack"
git diff | grep "print(" # 디버그용 출력문 제거 확인
git diff | grep "DEBUG = True" # 운영 설정 확인
git diff | grep "import pdb" # 디버거 호출부 확인
```

## 검증 리포트 구성 예시 (Report Template)

```
[DJANGO 검증 결과 보고서]
==========================

1단계: 환경 점검
  ✓ Python 3.11.5
  ✓ 가상 환경 활성화됨
  ✓ 환경 변수 주입 완료

2단계: 코드 품질
  ✓ mypy: 타입 오류 없음
  ✓ ruff: 3개 이슈 발견 (자동 수정 완료)
  ✓ black: 포맷팅 일치
  ✓ isort: 임포트 정렬 완료

3단계: 마이그레이션
  ✓ 미적용 마이그레이션 없음
  ✓ 충돌 없음
  ✓ 모든 모델 변경사항 반영됨

4단계: 테스트 및 커버리지
  결과: 247 통과, 0 실패, 5 건너뜀
  커버리지:
    - 전체: 87%
    - users: 92%
    - products: 89%

5단계: 보안 스캔
  ⚠ pip-audit: 취약한 패키지 2개 발견 (업데이트 필요)
  ✓ bandit: 보안 경고 없음
  ✓ 시크릿 유출 없음
  ✓ DEBUG = False 확인됨

배포 권고 사항: ⚠ 배포 전 pip-audit에서 발견된 취약 패키지를 반드시 업데이트하십시오.

다음 단계:
1. 취약점 발견된 패키지 버전 업그레이드
2. 보안 스캔 재실행
3. 스테이징 환경 배포 및 최종 웹 테스트
```

## 배포 전 최종 체크리스트

* [ ] 모든 테스트 통과 여부
* [ ] 전체 커버리지 80% 이상 달성 여부
* [ ] 알려진 보안 취약점 해결 여부
* [ ] 미적용 마이그레이션 파일 존재 여부
* [ ] 운영 설정에서 `DEBUG = False` 확인
* [ ] `SECRET_KEY`가 환경 변수로 안전하게 분리되었는가
* [ ] `ALLOWED_HOSTS`가 운영 도메인으로 정확히 설정되었는가
* [ ] 데이터베이스 백업 프로세스가 가동 중인가
* [ ] 정적 파일(Static files) 수집 및 서빙 설정 확인
* [ ] 에러 모니터링(Sentry 등) 작동 여부
* [ ] Redis/Cache 백언드 연결 상태 확인
* [ ] HTTPS/SSL 적용 및 강제 리다이렉트 확인

**기억하십시오**: 자동화된 검증 루프는 휴먼 에러를 줄여주지만, 스테이징 환경에서의 수동 코드 리뷰와 실제 사용자 시나리오 테스트를 완전히 대체할 수는 없습니다.
