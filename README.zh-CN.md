# Everything Claude Code (한국어 서비스 가이드)

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go&logoColor=white)

---

<div align="center">

**🌐 언어 / Language / 语言**

[English](README.md) | [**한국어**](README.zh-CN.md) | [简体中文](README.zh-CN.md) | [繁體中文](docs/zh-TW/README.md) | [日本語](docs/ja-JP/README.md)

</div>

---

**Anthropic 해커톤 우승자가 제공하는 Claude Code 설정의 완결판.**

실제 제품을 개발하며 10개월 이상의 집중적인 일상 사용을 통해 진화한 프로덕션 등급의 에이전트, 스킬, 후크, 명령어, 규칙 및 MCP 설정 모음입니다.

---

## 가이드

이 레포지토리는 원본 코드만을 포함합니다. 상세 가이드에서 모든 것을 설명합니다.

<table>
<tr>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" alt="Everything Claude Code 단축 가이드" />
</a>
</td>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" alt="Everything Claude Code 상세 가이드" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>단축 가이드</b><br/>설정, 기초, 철학. <b>이것을 먼저 읽으십시오.</b></td>
<td align="center"><b>상상세 가이드</b><br/>Token 최적화, 메모리 지속성, 평가, 병렬화.</td>
</tr>
</table>

| 주제 | 배울 내용 |
|-------|-------------------|
| Token 최적화 | 모델 선택, 시스템 프롬프트 간소화, 백그라운드 프로세스 |
| 메모리 지속성 | 세션 간에 컨텍스트를 자동으로 저장/로드하는 후크 |
| 지속적 학습 | 세션에서 패턴을 자동 추출하여 재사용 가능한 스킬로 변환 |
| 검증 루프 | 체크포인트 vs 지속적 평가, 채점자 유형, pass@k 지표 |
| 병렬화 | Git worktrees, 단계별(cascade) 방식, 인스턴스 확장 시기 |
| 서브에이전트 오케스트레이션 | 컨텍스트 문제, 반복적 검색 패턴 |

---

## 🚀 빠른 시작

2분 안에 바로 시작하기:

### 1단계: 플러그인 설치

```bash
# 마켓플레이스 추가
/plugin marketplace add affaan-m/everything-claude-code

# 플러그인 설치
/plugin install everything-claude-code@everything-claude-code
```

### 2단계: 규칙(Rules) 설치 (필수)

> ⚠️ **중요:** Claude Code 플러그인은 `rules`를 자동으로 배포할 수 없으므로 수동 설치가 필요합니다:

```bash
# 먼저 레포지토리 클론
git clone https://github.com/affaan-m/everything-claude-code.git

# 규칙 복사 (모든 프로젝트에 적용)
cp -r everything-claude-code/rules/* ~/.claude/rules/
```

### 3단계: 사용 시작

```bash
# 명령어 써보기 (플러그인 설치 시 네임스페이스 형식 사용)
/everything-claude-code:plan "사용자 인증 추가"

# 수동 설치(옵션 2) 시 짧은 형식 사용 가능:
# /plan "사용자 인증 추가"

# 사용 가능한 명령어 확인
/plugin list everything-claude-code@everything-claude-code
```

✨ **완료!** 이제 13개의 에이전트, 43개의 스킬, 31개의 명령어를 사용할 수 있습니다.

---

## 🌐 크로스 플랫폼 지원

이 플러그인은 **Windows, macOS, Linux**를 완벽하게 지원합니다. 모든 후크와 스크립트는 최대의 호환성을 위해 Node.js로 재작성되었습니다.

### 패키지 매니저 감지

플러그인은 다음과 같은 우선순위로 선호하는 패키지 매니저(npm, pnpm, yarn, bun)를 자동 감지합니다:

1. **환경 변수**: `CLAUDE_PACKAGE_MANAGER`
2. **프로젝트 설정**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 필드
4. **락(lock) 파일**: package-lock.json, yarn.lock, pnpm-lock.yaml, bun.lockb 감지
5. **글로벌 설정**: `~/.claude/package-manager.json`
6. **폴백(Fallback)**: 사용 가능한 첫 번째 패키지 매니저

선호하는 패키지 매니저를 설정하려면:

```bash
# 환경 변수를 통해
export CLAUDE_PACKAGE_MANAGER=pnpm

# 글로벌 설정을 통해
node scripts/setup-package-manager.js --global pnpm

# 프로젝트 설정을 통해
node scripts/setup-package-manager.js --project bun

# 현재 설정 감지
node scripts/setup-package-manager.js --detect
```

또는 Claude Code에서 `/setup-pm` 명령어를 사용하세요.

---

## 📦 내부 구성

이 레포지토리는 **Claude Code 플러그인**입니다. 직접 설치하거나 구성 요소를 수동으로 복사하여 사용할 수 있습니다.

```
everything-claude-code/
|-- .claude-plugin/   # 플러그인 및 마켓플레이스 매니페스트
|
|-- agents/           # 위임을 위한 전문 서브 에이전트
|   |-- planner.md           # 기능 구현 계획
|   |-- architect.md         # 시스템 설계 결정
|   |-- tdd-guide.md         # 테스트 주도 개발
|   |-- code-reviewer.md     # 품질 및 보안 리뷰
|
|-- skills/           # 워크플로우 정의 및 도메인 지식
|   |-- coding-standards/    # 언어별 최선 관행
|   |-- backend-patterns/    # API, DB, 캐싱 패턴
|   |-- frontend-patterns/   # React, Next.js 패턴
|
|-- commands/         # 빠른 실행을 위한 슬래시 명령어
|   |-- tdd.md              # /tdd - 테스트 주도 개발
|   |-- plan.md             # /plan - 구현 계획 수립
|
|-- rules/            # 항상 준수해야 할 가이드라인
|-- hooks/            # 트리거 기반 자동화
|-- scripts/          # 크로스 플랫폼 Node.js 스크립트
|-- mcp-configs/      # MCP 서버 구성
```

---

## 🛠️ 에코시스템 도구

### 스킬 생성기 (Skill Creator)

레포지토리에서 Claude Code 스킬을 생성하는 두 가지 방법:

#### 옵션 A: 로컬 분석 (기본 기능)

외부 서비스 없이 `/skill-create` 명령어로 로컬 분석을 수행합니다:

```bash
/skill-create                    # 현재 레포지토리 분석
/skill-create --instincts        # 지속적 학습을 위한 본능(instincts)도 생성
```

#### 옵션 B: GitHub 앱 (고급 기능)

[GitHub 앱 설치](https://github.com/apps/skill-creator) | [ecc.tools](https://ecc.tools)

- 커밋 히스토리에서 패턴 추출
- 자동으로 SKILL.md 파일 생성

---

## 🧠 지속적 학습 (Continuous Learning) v2

본능 기반 학습 시스템이 사용자의 패턴을 자동으로 학습합니다:

```bash
/instinct-status        # 학습된 본능과 신뢰도 확인
/instinct-import <file> # 타인으로부터 본능 가져오기
/instinct-export        # 내 본능을 공유용으로 내보내기
/evolve                 # 관련된 본능을 스킬로 묶기
```

---

## 📥 설치 방법

### 옵션 1: 플러그인으로 설치 (권장)

```bash
/plugin marketplace add affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

### 옵션 2: 수동 설치

```bash
git clone https://github.com/affaan-m/everything-claude-code.git
cp everything-claude-code/agents/*.md ~/.claude/agents/
cp everything-claude-code/rules/*.md ~/.claude/rules/
cp everything-claude-code/commands/*.md ~/.claude/commands/
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

---

## 🎯 핵심 개념

### 에이전트 (Agents)
제한된 범위 내에서 위임된 작업을 처리하는 서브 에이전트.

### 스킬 (Skills)
명령어나 에이전트에 의해 호출되는 워크플로우 정의.

### 후크 (Hooks)
도구(tool) 이벤트 발생 시 실행되는 자동화 작업. 가령 `console.log`가 있으면 경고를 띄울 수 있습니다.

### 규칙 (Rules)
항상 따라야 하는 가이드라인 (보안, 코딩 스타일, 테스트 등).

---

## 🤝 기여하기

**여러분의 기여를 환영합니다.**

유용한 에이전트, 스킬, 후크 또는 개선된 규칙이 있다면 기여해 주세요! 상세 내용은 [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하십시오.

---

## 🌟 별(Star) 히스토리

[![Star History Chart](https://api.star-history.com/svg?repos=affaan-m/everything-claude-code&type=Date)](https://star-history.com/#affaan-m/everything-claude-code&Date)

---

## 📄 라이선스

MIT - 자유롭게 사용하고, 필요에 따라 수정하며, 가능하면 기여해 주세요.

---

**도움이 되었다면 별(Star)을 눌러주세요. 가이드를 읽고 멋진 결과물을 만들어 보세요.**
    
