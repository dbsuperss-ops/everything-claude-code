**언어:** [English](README.md) | **한국어** | [繁體中文](docs/zh-TW/README.md)

# Everything Claude Code (ECC)

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![Forks](https://img.shields.io/github/forks/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/network/members)
[![Contributors](https://img.shields.io/github/contributors/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/graphs/contributors)
[![npm ecc-universal](https://img.shields.io/npm/dw/ecc-universal?label=ecc-universal%20weekly%20downloads&logo=npm)](https://www.npmjs.com/package/ecc-universal)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Python](https://img.shields.io/badge/-Python-3776AB?logo=python&logoColor=white)
![Go](https://img.shields.io/badge/-Go-00ADD8?logo=go&logoColor=white)

> **50K+ 별(Stars)** | **6K+ 포크(Forks)** | **30명 이상의 기여자** | **6개 언어 지원** | **Anthropic 해커톤 우승작**

---

<div align="center">

**🌐 언어 / Language / 语言**

[English](README.md) | [**한국어**](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](docs/zh-TW/README.md) | [日本語](docs/ja-JP/README.md)

</div>

---

**AI 에이전트 하네스(Harness)를 위한 성능 최적화 시스템. Anthropic 해커톤 우승자의 노하우가 담겨 있습니다.**

단순한 설정이 아닙니다. 스킬, 본능(Instincts), 메모리 최적화, 지속적 학습, 보안 스캔, 연구 우선 개발 등 완전한 시스템입니다. 실제 제품을 구축하며 10개월 이상의 집중적인 일상 사용을 통해 진화한 프로덕션 준비 완료 에이전트, 후크, 명령어, 규칙 및 MCP 구성을 제공합니다.

**Claude Code**, **Codex**, **Cowork** 및 기타 AI 에이전트 하네스에서 작동합니다.

---

## 가이드 (The Guides)

이 레포지토리는 원본 코드만을 포함합니다. 가이드에서 모든 것을 설명합니다.

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
<td align="center"><b>단축 가이드 (Shorthand Guide)</b><br/>설정, 기초, 철학. <b>이것을 먼저 읽으십시오.</b></td>
<td align="center"><b>상세 가이드 (Longform Guide)</b><br/>토큰 최적화, 메모리 지속성, 평가(Evals), 병렬화.</td>
</tr>
</table>

| 주제                        | 배울 내용                                                   |
| --------------------------- | ----------------------------------------------------------- |
| 토큰 최적화                 | 모델 선택, 시스템 프롬프트 슬리밍, 백그라운드 프로세스      |
| 메모리 지속성               | 세션 간에 컨텍스트를 자동으로 저장/로드하는 후크            |
| 지속적 학습                 | 세션에서 패턴을 자동 추출하여 재사용 가능한 스킬로 변환     |
| 검증 루프                   | 체크포인트 vs 지속적 평가, 채점자 유형, pass@k 지표         |
| 병렬화                      | Git worktrees, 캐스케이드(cascade) 방식, 인스턴스 확장 시기 |
| 서브에이전트 오케스트레이션 | 컨텍스트 문제, 반복적 검색 패턴                             |

---

## 새로운 소식

### v1.8.0 — 하네스 성능 시스템 (2026년 3월)

- **하네스 우선(Harness-first) 릴리스** — ECC는 이제 단순한 설정 팩이 아니라 에이전트 하네스 성능 시스템으로 명확하게 정의되었습니다.
- **후크 신뢰성 전면 개편** — SessionStart 루트 폴백, Stop 단계 세션 요약, 불안정한 인라인 한 줄 코드를 대체하는 스크립트 기반 후크.
- **후크 런타임 제어** — 후크 파일을 수정하지 않고도 `ECC_HOOK_PROFILE=minimal|standard|strict` 및 `ECC_DISABLED_HOOKS=...`를 통해 제어 가능.
- **새로운 하네스 명령어** — `/harness-audit`, `/loop-start`, `/loop-status`, `/quality-gate`, `/model-route`.

---

## 🚀 빠른 시작

2분 안에 시작하기:

### 1단계: 플러그인 설치

```bash
# 마켓플레이스 추가
/plugin marketplace add affaan-m/everything-claude-code

# 플러그인 설치
/plugin install everything-claude-code@everything-claude-code
```

### 2단계: 규칙(Rules) 설치 (필수)

> ⚠️ **중요:** Claude Code 플러그인은 `rules`를 자동으로 배포할 수 없습니다. 수동으로 설치해야 합니다:

```bash
# 레포지토리 먼저 클론
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# 설치 프로그램 사용 권장 (공통 + 언어별 규칙 안전하게 처리)
./install.sh typescript    # 또는 python 또는 golang
```

### 3단계: 사용 시작

```bash
# 명령어 시도 (플러그인 설치 시 네임스페이스 형태 사용)
/everything-claude-code:plan "사용자 인증 추가"

# 사용 가능한 명령어 확인
/plugin list everything-claude-code@everything-claude-code
```

✨ **끝입니다!** 이제 16개의 에이전트, 65개의 스킬, 40개의 명령어를 사용할 수 있습니다.

---

## 📦 내부 구성

이 레포지토리는 **Claude Code 플러그인**입니다 - 직접 설치하거나 구성 요소를 수동으로 복사할 수 있습니다.

```
everything-claude-code/
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

## 🗺️ 어떤 에이전트를 사용해야 하나요?

어디서부터 시작해야 할지 모르겠나요? 이 빠른 참조를 확인하세요:

| 작업 내용                 | 사용 명령어                    | 사용 에이전트        |
| ------------------------- | ------------------------------ | -------------------- |
| 새로운 기능 계획          | `/plan "Auth 추가"`          | planner              |
| 시스템 아키텍처 설계      | `/plan` + architect 에이전트 | architect            |
| 테스트 먼저 작성하며 코딩 | `/tdd`                       | tdd-guide            |
| 방금 작성한 코드 리뷰     | `/code-review`               | code-reviewer        |
| 빌드 에러 수정            | `/build-fix`                 | build-error-resolver |
| 엔드투엔드 테스트 실행    | `/e2e`                       | e2e-runner           |
| 보안 취약점 찾기          | `/security-scan`             | security-reviewer    |
| 데드 코드 제거            | `/refactor-clean`            | refactor-cleaner     |

---

## 토큰 최적화 (Token Optimization)

토큰 소비를 관리하지 않으면 Claude Code 사용 비용이 많이 들 수 있습니다. 이 설정은 품질을 저하시키지 않으면서 비용을 크게 줄여줍니다.

### 권장 설정

`~/.claude/settings.json`에 추가:

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  }
}
```

| 설정                                | 기본값 | 권장값           | 영향                                              |
| ----------------------------------- | ------ | ---------------- | ------------------------------------------------- |
| `model`                           | opus   | **sonnet** | 비용 약 60% 절감; 80% 이상의 코딩 작업 처리 가능  |
| `MAX_THINKING_TOKENS`             | 31,999 | **10,000** | 요청당 숨겨진 사고(thinking) 비용 약 70% 절감     |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 95     | **50**     | 더 이른 시점에 압축 수행 — 긴 세션에서 품질 향상 |

---

## 🔗 링크

- **단축 가이드 (여기서 시작):** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **상세 가이드 (고급):** [The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- **팔로우:** [@affaanmustafa](https://x.com/affaanmustafa)

---

## 📄 라이선스 (License)

MIT - 자유롭게 사용하고, 필요에 따라 수정하며, 가능하면 기여해 주십시오.

---

**도움이 되었다면 이 레포지토리에 별(Star)을 눌러주세요. 두 가이드를 모두 읽어보십시오. 멋진 것을 만들어 봅시다.**
