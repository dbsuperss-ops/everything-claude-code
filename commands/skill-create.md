---
이름: skill-create
설명: 로컬 git 히스토리를 분석하여 코딩 패턴을 추출하고 SKILL.md 파일을 생성합니다. Skill Creator GitHub App의 로컬 버전입니다.
허용된_도구: ["Bash", "Read", "Write", "Grep", "Glob"]
---

# /skill-create - 로컬 스킬 생성

저장소의 git 히스토리를 분석하여 코딩 패턴을 추출하고, 팀의 관행을 Claude에게 가르쳐 줄 수 있는 SKILL.md 파일을 생성합니다.

## 사용법

```bash
/skill-create                    # 현재 저장소 분석
/skill-create --commits 100      # 최근 100개의 커밋 분석
/skill-create --output ./skills  # 출력 디렉토리 지정
/skill-create --instincts        # continuous-learning-v2를 위한 본능(instincts)도 생성
```

## 주요 기능

1. **Git 히스토리 파싱** - 커밋, 파일 변경 사항, 패턴 분석
2. **패턴 감지** - 반복되는 워크플로우 및 컨벤션 식별
3. **SKILL.md 생성** - 유효한 Claude Code 스킬 파일 생성
4. **본능(Instincts) 생성 (선택 사항)** - continuous-learning-v2 시스템용

## 분석 단계

### 1단계: Git 데이터 수집

```bash
# 파일 변경 사항을 포함한 최근 커밋 목록 가져오기
git log --oneline -n ${COMMITS:-200} --name-only --pretty=format:"%H|%s|%ad" --date=short

# 파일별 커밋 빈도 확인
git log --oneline -n 200 --name-only | grep -v "^$" | grep -v "^[a-f0-9]" | sort | uniq -c | sort -rn | head -20

# 커밋 메시지 패턴 확인
git log --oneline -n 200 | cut -d' ' -f2- | head -50
```

### 2단계: 패턴 감지

다음과 같은 패턴 유형을 찾습니다:

| 패턴 | 감지 방법 |
|---------|-----------------|
| **커밋 컨벤션** | 커밋 메시지 정규식 (feat:, fix:, chore: 등) |
| **파일 동시 변경** | 항상 함께 변경되는 파일들 |
| **워크플로우 시퀀스** | 반복되는 파일 변경 패턴 |
| **아키텍처** | 폴더 구조 및 명명 규칙 |
| **테스트 패턴** | 테스트 파일 위치, 명명, 커버리지 |

### 3단계: SKILL.md 생성

출력 형식:

```markdown
---
name: {repo-name}-patterns
description: {repo-name}에서 추출된 코딩 패턴
version: 1.0.0
source: local-git-analysis
analyzed_commits: {count}
---

# {저장소 이름} 패턴

## 커밋 컨벤션
{감지된 커밋 메시지 패턴}

## 코드 아키텍처
{감지된 폴더 구조 및 조직 방식}

## 워크플로우
{감지된 반복적 파일 변경 패턴}

## 테스트 패턴
{감지된 테스트 컨벤션}
```

### 4단계: 본능(Instincts) 생성 (시작 시 --instincts 옵션 사용)

continuous-learning-v2 통합용:

```yaml
---
id: {repo}-commit-convention
trigger: "커밋 메시지를 작성할 때"
confidence: 0.8
domain: git
source: local-repo-analysis
---

# 시맨틱 커밋 사용 (Use Conventional Commits)

## 행동 (Action)
커밋 메시지 앞에 다음 접두사를 붙입니다: feat:, fix:, chore:, docs:, test:, refactor:

## 근거
- {n}개의 커밋 분석 완료
- {percentage}%가 시맨틱 커밋 형식을 따름
```

## 출력 예시

TypeScript 프로젝트에서 `/skill-create`를 실행하면 다음과 같이 생성될 수 있습니다:

```markdown
---
name: my-app-patterns
description: my-app 저장소에서 추출된 코딩 패턴
version: 1.0.0
source: local-git-analysis
analyzed_commits: 150
---

# My App 패턴

## 커밋 컨벤션

이 프로젝트는 **시맨틱 커밋(Conventional Commits)**을 사용합니다:
- `feat:` - 새로운 기능
- `fix:` - 버그 수정
- `chore:` - 유지보수 작업
- `docs:` - 문서 업데이트

## 코드 아키텍처

```
src/
├── components/     # React 컴포넌트 (PascalCase.tsx)
├── hooks/          # 커스텀 훅 (use*.ts)
├── utils/          # 유틸리티 함수
├── types/          # TypeScript 타입 정의
└── services/       # API 및 외부 서비스
```

## 워크플로우

### 새로운 컴포넌트 추가
1. `src/components/ComponentName.tsx` 생성
2. `src/components/__tests__/ComponentName.test.tsx`에 테스트 추가
3. `src/components/index.ts`에서 내보내기(export) 추가

### 데이터베이스 마이그레이션
1. `src/db/schema.ts` 수정
2. `pnpm db:generate` 실행
3. `pnpm db:migrate` 실행

## 테스트 패턴

- 테스트 파일: `__tests__/` 디렉토리 또는 `.test.ts` 접미사 사용
- 커버리지 목표: 80% 이상
- 프레임워크: Vitest
```

## GitHub App 통합

더 강력한 기능(1만 개 이상의 커밋 분석, 팀 공유, 자동 PR 등)이 필요한 경우 [Skill Creator GitHub App](https://github.com/apps/skill-creator)을 사용하십시오:

- 설치: [github.com/apps/skill-creator](https://github.com/apps/skill-creator)
- 이슈에 `/skill-creator analyze` 댓글 작성
- 자동 생성된 스킬이 포함된 PR 수신

## 관련 명령어

- `/instinct-import` - 생성된 본능 가져오기
- `/instinct-status` - 학습된 본능 확인
- `/evolve` - 본능을 스킬/에이전트로 클러스터링(Grouping)
