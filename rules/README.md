# 규칙 (Rules)

## 구조 (Structure)

규칙은 **공통(Common)** 레이어와 **언어별(Language-specific)** 디렉토리로 구성됩니다:

```
rules/
├── common/          # 언어에 무관한 원칙 (항상 설치)
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
├── swift/           # Swift 전용
└── php/             # PHP 전용
```

- **common/**: 모든 언어에 보편적으로 적용되는 원칙을 담고 있으며, 언어 전용 코드 예제는 포함하지 않습니다.
- **언어 전용 디렉토리**: 프레임워크별 패턴, 도구, 코드 예제를 통해 공통 규칙을 확장합니다. 각 파일은 대응하는 공통 파일을 참조합니다.

## 설치 방법 (Installation)

### 방법 1: 설치 스크립트 사용 (권장)

```bash
# 공통 규칙 + 하나 이상의 언어 전용 규칙 세트 설치
./install.sh typescript
./install.sh python
./install.sh golang
./install.sh swift
./install.sh php

# 여러 언어를 동시에 설치
./install.sh typescript python
```

### 방법 2: 직접 설치

> **중요:** 디렉토리 전체를 복사하십시오. `/*`를 사용하여 내용을 평면적으로(Flatten) 복사하지 마십시오.
> 공통 디렉토리와 언어 전용 디렉토리에는 이름이 같은 파일들이 포함되어 있습니다.
> 한 디렉토리에 모두 쏟아부으면 언어 전용 파일이 공통 규칙을 덮어쓰게 되며, 언어 전용 파일에서 사용하는 상대 경로(`../common/`) 참조가 깨지게 됩니다.

```bash
# 공통 규칙 설치 (모든 프로젝트에 필수)
cp -r rules/common ~/.claude/rules/common

# 프로젝트 기술 스택에 맞는 언어별 규칙 설치
cp -r rules/typescript ~/.claude/rules/typescript
cp -r rules/python ~/.claude/rules/python
cp -r rules/golang ~/.claude/rules/golang
cp -r rules/swift ~/.claude/rules/swift
cp -r rules/php ~/.claude/rules/php

# 주의: 실제 프로젝트 요구 사항에 맞게 구성하십시오. 위 설정은 참고용입니다.
```

## 규칙 vs 스킬 (Rules vs Skills)

- **규칙 (Rules)**: 광범위하게 적용되는 표준, 컨벤션, 체크리스트를 정의합니다. (예: "테스트 커버리지 80% 유지", "시크릿 하드코딩 금지")
- **스킬 (Skills)**: 특정 작업에 대한 구체적이고 실행 가능한 참조 자료를 제공합니다. (예: `python-patterns`, `golang-testing`)

언어별 규칙 파일은 적절한 경우 관련 스킬을 참조합니다. 규칙은 *무엇을(What)* 해야 하는지를 알려주고, 스킬은 *어떻게(How)* 하는지를 알려줍니다.

## 새로운 언어 추가 방법

새로운 언어(예: `rust/`) 지원을 추가하려면:

1. `rules/rust/` 디렉토리를 생성합니다.
2. 공통 규칙을 확장하는 파일들을 추가합니다:
   - `coding-style.md`: 포맷팅 도구, 관용구, 에러 처리 패턴
   - `testing.md`: 테스트 프레임워크, 커버리지 도구, 테스트 구성
   - `patterns.md`: 언어별 디자인 패턴
   - `hooks.md`: 포맷터, 린터, 타입 체크를 위한 PostToolUse 후크
   - `security.md`: 시크릿 관리, 보안 스캔 도구
3. 각 파일은 다음과 같이 시작해야 합니다:
   ```
   > 이 파일은 [common/xxx.md](../common/xxx.md)의 내용을 <언어> 전용 콘텐츠로 확장합니다.
   ```
4. 이미 존재하는 스킬을 참조하거나 `skills/` 아래에 새로운 스킬을 생성하십시오.

## 규칙 우선순위 (Rule Priority)

언어별 규칙과 공통 규칙이 충돌하는 경우, **언어별 규칙이 우선**합니다 (구체적인 것이 일반적인 것을 덮어씀). 이는 CSS 명시성이나 `.gitignore` 우선순위와 유사한 계층적 구성 패턴을 따릅니다.

- `rules/common/`: 모든 프로젝트에 적용되는 범용 기본값을 정의합니다.
- `rules/golang/`, `rules/python/` 등: 언어 특성에 따라 기본값이 다른 경우 이를 재정의(Override)합니다.

### 예시

`common/coding-style.md`에서 불변성(Immutability)을 기본 원칙으로 권장하더라도, 언어별 규칙인 `golang/coding-style.md`에서 이를 재정의할 수 있습니다:

> Go 관용구는 구조체 수정을 위해 포인터 리시버를 사용합니다. 일반적인 원칙은 [common/coding-style.md](../common/coding-style.md)를 참고하되, 여기서는 Go 관용적인 수정 방식을 우선합니다.

### 재정의 참고 사항이 있는 공통 규칙

언어별 파일에 의해 재정의될 수 있는 `rules/common/` 내의 규칙은 다음과 같이 표시됩니다:

> **언어 참고**: 이 패턴이 관용적이지 않은 언어의 경우, 언어별 규칙에 의해 재정의될 수 있습니다.
