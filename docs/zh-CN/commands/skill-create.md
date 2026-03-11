---
name: skill-create
description: 로컬 Git 히스토리를 분석하여 코딩 패턴을 추출하고 SKILL.md 파일을 생성합니다. Skill Creator GitHub 애플리케이션의 로컬 버전입니다.
allowed_tools: ["Bash", "Read", "Write", "Grep", "Glob"]
---

# /skill-create - 로컬 기반 스킬 생성

저장소의 Git 이력을 분석하여 팀 특유의 코딩 패턴을 추출하고, 이를 Claude가 학습할 수 있도록 `SKILL.md` 파일로 변환합니다.

## 사용법

```bash
/skill-create                    # 현재 저장소 분석
/skill-create --commits 100      # 최근 100개의 커밋 분석
/skill-create --output ./skills  # 출력 디렉토리 지정
/skill-create --instincts        # continuous-learning-v2용 본능(Instincts)도 함께 생성
```

## 주요 기능

1. **Git 히스토리 파싱**: 커밋 메시지, 수정 파일 목록, 변경 패턴을 분석합니다.
2. **패턴 감지**: 반복되는 워크플로우, 네이밍 컨벤션, 아키텍처 구조를 식별합니다.
3. **SKILL.md 생성**: Claude Code가 이해할 수 있는 형식의 스킬 파일을 작성합니다.
4. **본능(Instincts) 생성 (옵션)**: Continuous Learning v2 시스템에서 즉시 활용 가능한 행동 지침을 만듭니다.

---

## 분석 대상 패턴

| 패턴 유형 | 분석 내용 |
|---------|-----------------|
| **커밋 컨벤션** | 커밋 메시지 형식 (예: feat:, fix:, chore:) 식별 |
| **파일 동시 변경** | 항상 함께 수정되는 파일 세트(예: 스키마와 타입 정의) 파악 |
| **워크플로우 시퀀스** | 반복되는 파일 수정 순서 및 작업 그룹 감지 |
| **아키텍처 구조** | 폴더 구조 및 모듈화 방식 분석 |
| **테스트 패턴** | 테스트 파일 위치, 명명 규칙, 프레임워크 사용 방식 검축 |

---

## 실행 예시 (TypeScript 프로젝트)

`/skill-create` 실행 시 다음과 같은 `SKILL.md`가 생성될 수 있습니다:

```markdown
---
name: my-app-patterns
description: my-app 저장소에서 추출된 코딩 패턴
version: 1.0.0
source: local-git-analysis
analyzed_commits: 150
---

# My App 코딩 패턴 가이드

## 1. 커밋 규정
프로젝트는 **Conventional Commits**를 따릅니다:
- `feat:`: 새로운 기능 추가
- `fix:`: 버그 수정
- `docs:`: 문서 수정

## 2. 코드 아키텍처
```text
src/
├── components/     # React 컴포넌트 (PascalCase.tsx)
├── hooks/          # 커스텀 훅 (use*.ts)
├── types/          # TypeScript 타입 정의 전용
└── services/       # API 호출 로직
```

## 3. 주요 워크플로우
### 새 컴포넌트 추가 시
1. `src/components/ComponentName.tsx` 생성
2. `src/components/__tests__/` 경로에 테스트 코드 작성
3. `src/components/index.ts`를 통해 외부에 노출(Export)

## 4. 테스트 컨벤션
- Vitest 프레임워크 사용
- 테스트 파일은 `.test.ts` 접미사 사용 또는 `__tests__/` 폴더 내 위치
- 목표 커버리지: 80% 이상
```

---

## 고급 기능: GitHub 앱 통합

대형 프로젝트(10,000개 이상의 커밋)나 팀 단위 공유가 필요한 경우, 자동 PR 기능이 포함된 [Skill Creator GitHub App](https://github.com/apps/skill-creator) 사용을 권장합니다.

**관련 명령어**:
* `/instinct-import`: 생성된 본능(instincts)을 시스템에 로컬 적용
* `/instinct-status`: 현재 학습된 본능 현황 확인
* `/evolve`: 학습된 본능들을 모아 더 고도화된 스킬이나 에이전트로 진화

**핵심**: Skill-Create 명령어는 팀의 암묵적인 지식(커밋 히스토리)을 명시적인 에이전트 지침(SKILL.md)으로 변환하여 기술 부채를 줄이고 협업 효율을 높입니다.
