---
name: security-scan
description: AgentShield를 사용하여 Claude Code 설정(.claude/ 디렉토리)의 보안 취약점, 오설정 및 인젝션 리스크를 스캔합니다. CLAUDE.md, settings.json, MCP 서버, 후크 및 에이전트 정의를 점검합니다.
origin: ECC
---

# 보안 스캔 스킬 (Security Scan Skill)

[AgentShield](https://github.com/affaan-m/agentshield)를 사용하여 Claude Code 설정의 보안 문제를 감사(Audit)합니다.

## 활성화 시점

- 새로운 Claude Code 프로젝트를 설정할 때
- `.claude/settings.json`, `CLAUDE.md`, 또는 MCP 설정을 수정한 후
- 설정 변경 사항을 커밋하기 전
- 기존 Claude Code 설정이 있는 새로운 레포지토리에 온보딩할 때
- 정기적인 보안 위생 점검 시

## 스캔 대상 및 항목

- **CLAUDE.md**: 하드코딩된 시크릿, 자동 실행 지침, 프롬프트 인젝션 패턴.
- **settings.json**: 과도하게 허용된 화이트리스트, 누락된 차단 리스트(Deny list), 위험한 우회 플래그.
- **mcp.json**: 위험한 MCP 서버, 하드코딩된 환경 변수 시크릿, npx 공급망 리스크.
- **hooks/**: 보간(Interpolation)을 통한 명령 인젝션, 데이터 유출, 조용한 에러 억제.
- **agents/*.md**: 무제한 도구 액세스, 프롬프트 인젝션 노출면, 모델 사양 누락.

## 주요 사용법

### 1. 기본 스캔
- `npx ecc-agentshield scan` 명령으로 현재 프로젝트를 스캔합니다.
- 특정 경로 지정(`--path`)이나 최소 심각도 필터링(`--min-severity`)이 가능합니다.

### 2. 출력 형식 및 자동 수정
- 터미널(기본), JSON, 마크다운, HTML 형식을 지원합니다.
- `--fix` 옵션을 사용하여 환경 변수 참조 교체 등 자동 수정 가능한 항목을 처리할 수 있습니다.

### 3. 심층 분석 (Opus 4.6 Deep Analysis)
- `--opus --stream` 옵션을 사용하여 공격자(Red Team), 방어자(Blue Team), 감사자(Auditor)로 구성된 3단계 에이전트 파이프라인으로 더 깊은 분석을 수행합니다. (ANTHROPIC_API_KEY 필요)

### 4. 설정 초기화 및 CI 통합
- `npx ecc-agentshield init`으로 보안 최선 관행이 적용된 기본 설정을 생성할 수 있습니다.
- GitHub Actions를 통해 CI 파이프라인에서 자동으로 스캔을 수행할 수 있습니다.

## 결과 해석 (위험도별)

- **Critical (즉시 수정)**: 하드코딩된 API 키, 무제한 쉘 액세스(`Bash(*)`), 후크 내 명령 인젝션 취약점 등.
- **High (운영 전 수정)**: CLAUDE.md 내 자동 실행 지침, 누락된 차단 리스트 등.
- **Medium (권장)**: 조용한 에러 억제(`2>/dev/null` 등), MCP 설정의 `npx -y` 자동 설치 등.

**기억하십시오**: 보안 스캔을 통해 에이전트가 예상치 못한 위험한 동작을 수행하지 않도록 설정을 견고히 관리하십시오.
    
