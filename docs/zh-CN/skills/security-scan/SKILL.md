---
name: security-scan
description: AgentShield를 사용하여 Claude Code 설정(.claude/ 디렉터리)을 스캔하고 보안 취약점, 설정 오류 및 인젝션 위험을 발견합니다. CLAUDE.md, settings.json, MCP 서버, 훅(Hooks), 에이전트 정의를 점검합니다.
origin: ECC
---

# 보안 스캔 스킬 (Security Scan)

[AgentShield](https://github.com/affaan-m/agentshield)를 사용하여 Claude Code 설정의 보안 문제를 감사하십시오.

## 적용 시점

* 새로운 Claude Code 프로젝트를 시작할 때
* `.claude/settings.json`, `CLAUDE.md` 또는 MCP 설정을 변경한 후
* 설정 변경을 커밋하기 전
* 기존 Claude Code 설정이 있는 새로운 코드베이스를 맡았을 때
* 정기적인 보안 상태 점검 시

## 스캔 대상 항목

| 파일/경로 | 주요 점검 내용 |
|------|--------|
| `CLAUDE.md` | 하드코딩된 비밀 정보, 자동 실행 명령, 프롬프트 인젝션 패턴 |
| `settings.json` | 너무 관대한 허용 목록, 누락된 거부 목록, 위험한 우회 플래그 |
| `mcp.json` | 위험한 MCP 서버, 환경 변수 내 비밀 정보, npx 공급망 리스크 |
| `hooks/` | `${file}` 보간법을 통한 커맨드 인젝션, 데이터 유출, 자동 오류 무시 |
| `agents/*.md` | 무제한 도구 접근, 프롬프트 인젝션 노출면, 모델 사양 누락 |

## 사전 준비 (AgentShield 설치)

AgentShield가 설치되어 있어야 합니다. 필요 시 설치하십시오:

```bash
# 직접 실행 (설정 없이 npx로 실행 가능)
npx ecc-agentshield scan .
```

## 사용 방법

### 1. 기본 스캔
현재 프로젝트의 `.claude/` 디렉터리를 대상으로 실행합니다:

```bash
# 현재 프로젝트 스캔
npx ecc-agentshield scan

# 특정 경로 스캔
npx ecc-agentshield scan --path /path/to/.claude

# 최소 심각도 필터 적용 (예: medium 이상)
npx ecc-agentshield scan --min-severity medium
```

### 2. 출력 형식 선택
```bash
# 터미널 컬러 리포트 (기본값)
npx ecc-agentshield scan

# JSON 형식 (CI/CD 통합용)
npx ecc-agentshield scan --format json

# Markdown 형식 (문서화용)
npx ecc-agentshield scan --format markdown

# HTML 형식 (다크 테마 리포트 파일 생성)
npx ecc-agentshield scan --format html > security-report.html
```

### 3. 자동 수정 (Auto-fix)
자동 수정이 가능한 문제들을 즉시 반영합니다:

```bash
npx ecc-agentshield scan --fix
```
* 하드코딩된 비밀 정보를 환경 변수 참조로 교체합니다.
* 와일드카드(`*`) 권한을 구체적인 범위로 좁힙니다.

## 주요 등급 및 지표

| 등급 | 점수 | 의미 |
|-------|-------|---------|
| A | 90-100 | 보안 설정 우수 |
| B | 75-89 | 경미한 문제 발견 |
| C | 60-74 | 주의 필요 |
| D | 40-59 | 상당한 위험 존재 |
| F | 0-39 | 심각한 보안 취약점 |

### 🚨 즉시 수정해야 할 주요 발견 사항
* 설정 파일 내 하드코딩된 API 키 또는 토큰
* 허용 목록의 `Bash(*)` (제한 없는 쉘 접근 권한)
* 훅(Hooks) 내 `${file}` 보간을 통한 커맨드 인젝션 취약점
* 쉘을 직접 실행하는 MCP 서버 설정

**핵심**: 보안 스캔을 통해 Claude Code를 더욱 안전하게 사용하십시오. 자동화된 도구를 활용하여 설정 단계에서의 실수와 취약점을 사전에 방어하십시오.
