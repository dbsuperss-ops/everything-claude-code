# 규칙 (Rules)

## 구조

규칙은 **공통(Common)** 레이어와 **언어별(Language-specific)** 디렉토리로 구성됩니다:

```
rules/
├── common/          # 언어 중립적인 원칙 (항상 설치)
10: │   ├── coding-style.md
11: │   ├── git-workflow.md
12: │   ├── testing.md
13: │   ├── performance.md
14: │   ├── patterns.md
15: │   ├── hooks.md
16: │   ├── agents.md
17: │   └── security.md
├── typescript/      # TypeScript/JavaScript 전용
├── python/          # Python 전용
├── golang/          # Go 전용
└── swift/           # Swift 전용
```

* **common/** 디렉토리는 특정 언어에 국한되지 않는 범용적인 원칙을 포함하며, 언어별 코드 예제는 포함하지 않습니다.
* **언어별 디렉토리**는 프레임워크별 패턴, 도구 및 코드 예제를 통해 공통 규칙을 확장합니다. 각 파일은 해당되는 공통 파일을 참조합니다.

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

> **중요:** 디렉토리 전체를 복사하십시오. `/*`를 사용하여 구조를 평면화(flatten)하지 마세요.
> 공통 디렉토리와 언어별 디렉토리는 동일한 이름의 파일들을 포함하고 있습니다.
> 이들을 하나의 디렉토리에 몰아넣으면 언어별 파일이 공통 규칙 파일을 덮어쓰게 되며, 언어별 파일에서 사용하는 `../common/` 상대 경로 참조가 깨지게 됩니다.

```bash
# 공통 규칙 설치 (모든 프로젝트에 필요)
cp -r rules/common ~/.claude/rules/common

# 프로젝트의 기술 스택에 맞춰 언어별 규칙 설치
cp -r rules/typescript ~/.claude/rules/typescript
cp -r rules/python ~/.claude/rules/python
cp -r rules/golang ~/.claude/rules/golang
cp -r rules/swift ~/.claude/rules/swift

# 주의! 실제 프로젝트 요구 사항에 맞춰 구성하십시오. 여기에 설명된 설정은 참고용일 뿐입니다.
```

## 규칙(Rules)과 스킬(Skills)

* **규칙**은 광범위하게 적용되는 표준, 컨벤션 및 체크리스트를 정의합니다 (예: "테스트 커버리지 80%", "하드코딩된 비밀 키 금지").
* **스킬**(`skills/` 디렉토리)은 특정 작업에 대한 심층적이고 실행 가능한 참조 자료를 제공합니다 (예: `python-patterns`, `golang-testing`).

언어별 규칙 파일은 적절한 위치에서 관련 스킬을 참조합니다. 규칙은 *무엇을 해야 하는지* 알려주고, 스킬은 *어떻게 하는지* 알려줍니다.

## 새로운 언어 추가

새로운 언어(예: `rust/`)에 대한 지원을 추가하려면:

1. `rules/rust/` 디렉토리를 생성합니다.
2. 공통 규칙을 확장하는 파일들을 추가합니다:
   * `coding-style.md` —— 포매팅 도구, 관용구, 에러 처리 패턴
   * `testing.md` —— 테스트 프레임워크, 커버리지 도구, 테스트 구성
   * `patterns.md` —— 언어별 디자인 패턴
   * `hooks.md` —— 포매터, 린터, 타입 체크용 PostToolUse 후크
   * `security.md` —— 비밀 키 관리, 보안 스캔 도구
3. 각 파일은 다음 내용으로 시작해야 합니다:
   ```
   > 이 파일은 <언어> 관련 내용으로 [common/xxx.md](../common/xxx.md)의 내용을 확장합니다.
   ```
4. 기존 스킬이 있다면 참조하고, 없다면 `skills/` 아래에 새로운 스킬을 생성합니다.

## 규칙 우선순위

언어별 규칙과 공통 규칙이 충돌할 경우, **언어별 규칙이 우선**합니다 (구체적인 규칙이 일반적인 규칙을 덮어씀). 이는 표준 계층형 구성 패턴(CSS 명시도나 `.gitignore` 우선순위와 유사함)을 따릅니다.

* `rules/common/`은 모든 프로젝트에 적용되는 일반적인 기본값을 정의합니다.
* `rules/golang/`, `rules/python/`, `rules/typescript/` 등은 언어적 관례가 다른 부분에서 이러한 기본값을 덮어씁니다.

### 예시

`common/coding-style.md`는 기본적으로 불변성(Immutability)을 원칙으로 권장합니다. 하지만 언어별 `golang/coding-style.md`에서는 이를 다음과 같이 덮어쓸 수 있습니다:

> Go 언어의 관례에 따라 구조체 수정 시 포인터 리시버를 사용합니다. 일반적인 원칙은 [common/coding-style.md](../../../common/coding-style.md)를 참조하되, 여기서는 Go의 관례에 따른 수정 방식을 권장합니다.

### 덮어쓰기 설명이 포함된 공통 규칙

`rules/common/` 파일들 중 언어별 파일에 의해 덮어써질 가능성이 있는 규칙은 다음과 같이 표시됩니다:

> **언어별 참고**: 이 패턴이 해당 언어의 관례와 맞지 않는 경우, 언어별 규칙에 의해 덮어써질 수 있습니다.
