# 규칙 (Rules)
## 구조

규칙은 **공통(common)** 레이어와 **언어별(language-specific)** 디렉토리로 구성됩니다:

```
rules/
├── common/          # 언어 중립적인 원칙 (항상 설치)
│   ├── coding-style.md
│   ├── git-workflow.md
│   ├── testing.md
│   ├── performance.md
│   ├── patterns.md
│   ├── hooks.md
│   ├── agents.md
│   └── security.md
├── typescript/      # TypeScript/JavaScript 전용
├── python/          # Python 전용
├── golang/          # Go 전용
└── swift/           # Swift 전용
```

- **common/**은 범용적인 원칙을 포함하며, 언어별 코드 예제는 포함하지 않습니다.
- **언어별 디렉토리**는 프레임워크 전용 패턴, 도구 및 코드 예제와 함께 공통 규칙을 확장합니다. 각 파일은 해당 공통 파일을 참조합니다.

## 설치

### 옵션 1: 설치 스크립트 (권장)

```bash
# 공통 규칙 + 하나 이상의 언어별 규칙 세트 설치
./install.sh typescript
./install.sh python
./install.sh golang
./install.sh swift

# 여러 언어를 한 번에 설치
./install.sh typescript python
```

### 옵션 2: 수동 설치

> **중요:** 디렉토리 전체를 복사하십시오. `/*`를 사용하여 내용을 직접 펼치지(flatten) 마십시오.
> 공통 디렉토리와 언어별 디렉토리에는 이름이 같은 파일들이 포함되어 있습니다.
> 이를 하나의 디렉토리에 펼쳐서 복사하면 언어별 파일이 공통 규칙을 덮어쓰게 되며, 
> 언어별 파일에서 사용하는 상대 경로(`../common/`) 참조가 깨지게 됩니다.

```bash
# 공통 규칙 설치 (모든 프로젝트에 필요)
cp -r rules/common ~/.claude/rules/common

# 프로젝트의 기술 스택에 따른 언어별 규칙 설치
cp -r rules/typescript ~/.claude/rules/typescript
cp -r rules/python ~/.claude/rules/python
cp -r rules/golang ~/.claude/rules/golang
cp -r rules/swift ~/.claude/rules/swift

# 주의! ! ! 실제 프로젝트 요구 사항에 따라 구성하십시오. 여기의 구성은 참조용일 뿐입니다.
```

## 규칙(Rules) vs 스킬(Skills)

- **규칙(Rules)**은 광범위하게 적용되는 표준, 컨벤션 및 체크리스트를 정의합니다 (예: "80% 테스트 커버리지", "하드코딩된 비밀 정보 금지").
- **스킬(Skills)** (`skills/` 디렉토리)은 특정 작업을 위한 깊이 있고 실행 가능한 참조 자료를 제공합니다 (예: `python-patterns`, `golang-testing`).

언어별 규칙 파일은 적절한 경우 관련 스킬을 참조합니다. 규칙은 *무엇(what)*을 해야 하는지 알려주고, 스킬은 *어떻게(how)* 해야 하는지 알려줍니다.

## 새로운 언어 추가

새로운 언어(예: `rust/`)에 대한 지원을 추가하려면:

1. `rules/rust/` 디렉토리를 생성합니다.
2. 공통 규칙을 확장하는 파일을 추가합니다:
   - `coding-style.md` — 포매팅 도구, 관용구, 에러 처리 패턴
   - `testing.md` — 테스트 프레임워크, 커버리지 도구, 테스트 구성
   - `patterns.md` — 언어별 디자인 패턴
   - `hooks.md` — 포매터, 린터, 타입 체크를 위한 PostToolUse 훅
   - `security.md` — 비밀 정보 관리, 보안 스캔 도구
3. 각 파일은 다음과 같이 시작해야 합니다:
   ```
   > 이 파일은 [common/xxx.md](../common/xxx.md)을 <언어> 전용 내용으로 확장합니다.
   ```
4. 이미 있는 경우 기존 스킬을 참조하거나, `skills/` 아래에 새로운 스킬을 생성하십시오.

## 규칙 우선순위

언어별 규칙과 공통 규칙이 충돌할 경우, **언어별 규칙이 우선**합니다 (구체적인 것이 일반적인 것을 덮어씀). 이는 표준적인 레이어드 구성 패턴(CSS 명시도나 `.gitignore` 우선순위와 유사함)을 따릅니다.

- `rules/common/`은 모든 프로젝트에 적용되는 범용 기본값을 정의합니다.
- `rules/golang/`, `rules/python/`, `rules/typescript/` 등은 언어별 관용구가 다를 경우 해당 기본값을 덮어씁니다.

### 예시

`common/coding-style.md`는 기본 원칙으로 불변성을 권장합니다. 언어별 `golang/coding-style.md`는 이를 덮어쓸 수 있습니다:

> 관용적인 Go는 구조체 수정을 위해 포인터 리시버를 사용합니다. 일반적인 원칙은 [common/coding-style.md](../common/coding-style.md)를 참조하되, 여기서는 Go의 관용적 수정을 선호합니다.

### 재정의 노트가 포함된 공통 규칙

언어별 파일에 의해 재정의될 수 있는 `rules/common/`의 규칙들은 다음과 같이 표시되어 있습니다:

> **언어 참고**: 이 패턴이 관용적이지 않은 언어의 경우, 언어별 규칙에 의해 이 규칙이 재정의될 수 있습니다.
