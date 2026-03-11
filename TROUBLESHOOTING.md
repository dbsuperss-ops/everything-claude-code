# 문제 해결 가이드 (Troubleshooting Guide)

Everything Claude Code (ECC) 플러그인에서 발생하는 일반적인 문제와 해결 방법입니다.

## 목차

- [메모리 및 컨텍스트 문제](#메모리-및-컨텍스트-문제)
- [에이전트 하네스 실패](#에이전트-하네스-실패)
- [후크 및 워크플로우 오류](#후크-및-워크플로우-오류)
- [설치 및 설정](#설치-및-설정)
- [성능 문제](#성능-문제)
- [일반적인 에러 메시지](#일반적인-에러-메시지)
- [도움 받기](#도움-받기)

---

## 메모리 및 컨텍스트 문제

### 컨텍스트 윈도우 오버플로우 (Context Window Overflow)

**증상:** "Context too long" 에러 발생 또는 불완전한 응답

**원인:**
- 토큰 제한을 초과하는 대용량 파일 업로드
- 누적된 대화 기록
- 단일 세션 내의 다중 대용량 도구 출력

**해결 방법:**
```bash
# 1. 대화 기록을 지우고 새로 시작하십시오.
# Claude Code: "New Chat" 또는 Cmd/Ctrl+Shift+N 사용

# 2. 분석 전 파일 크기를 줄이십시오.
head -n 100 large-file.log > sample.log

# 3. 대용량 출력에는 스트리밍을 사용하십시오.
head -n 50 large-file.txt

# 4. 작업을 작은 단위로 나누십시오.
# "50개 파일 전체 분석" 대신
# "src/components/ 디렉토리 내 파일 분석"으로 요청하십시오.
```

### 메모리 지속성 실패 (Memory Persistence Failures)

**증상:** 에이전트가 이전 컨텍스트나 관찰 내용을 기억하지 못함

**원인:**
- continuous-learning 후크 비활성화
- 손상된 관찰(observation) 파일
- 프로젝트 감지 실패

**해결 방법:**
```bash
# 관찰 내용이 기록되고 있는지 확인하십시오.
ls ~/.claude/homunculus/projects/*/observations.jsonl

# 현재 프로젝트의 해시 ID를 찾으십시오.
python3 - <<'PY'
import json, os
registry_path = os.path.expanduser("~/.claude/homunculus/projects.json")
with open(registry_path) as f:
    registry = json.load(f)
for project_id, meta in registry.items():
    if meta.get("root") == os.getcwd():
        print(project_id)
        break
else:
    raise SystemExit("Project hash not found in ~/.claude/homunculus/projects.json")
PY

# 해당 프로젝트의 최근 관찰 내용을 확인하십시오.
tail -20 ~/.claude/homunculus/projects/<project-hash>/observations.jsonl

# 손상된 관찰 파일을 재생성하기 전 백업하십시오.
mv ~/.claude/homunculus/projects/<project-hash>/observations.jsonl \
  ~/.claude/homunculus/projects/<project-hash>/observations.jsonl.bak.$(date +%Y%m%d-%H%M%S)

# 후크가 활성화되어 있는지 확인하십시오.
grep -r "observe" ~/.claude/settings.json
```

---

## 에이전트 하네스 실패

### 에이전트를 찾을 수 없음 (Agent Not Found)

**증상:** "Agent not loaded" 또는 "Unknown agent" 에러 발생

**원인:**
- 플러그인이 올바르게 설치되지 않음
- 에이전트 경로 설정 오류
- 마켓플레이스 설치와 수동 설치 간의 불일치

**해결 방법:**
```bash
# 플러그인 설치 확인
ls ~/.claude/plugins/cache/

# 에이전트 존재 확인 (마켓플레이스 설치 시)
ls ~/.claude/plugins/cache/*/agents/

# 수동 설치의 경우, 에이전트 위치 확인:
ls ~/.claude/agents/  # 커스텀 에이전트 전용

# 플러그인 재로드
# Claude Code → Settings → Extensions → Reload
```

### 워크플로우 실행 중단 (Workflow Execution Hangs)

**증상:** 에이전트가 시작되었으나 완료되지 않음

**원인:**
- 에이전트 로직 내 무한 루프
- 사용자 입력 대기 중 차단됨
- API 대기 중 네트워크 타임아웃

**해결 방법:**
```bash
# 1. 멈춘 프로세스 확인
ps aux | grep claude

# 2. 디버그 모드 활성화
export CLAUDE_DEBUG=1

# 3. 짧은 타임아웃 설정
export CLAUDE_TIMEOUT=30

# 4. 네트워크 연결 확인
curl -I https://api.anthropic.com
```

### 도구 사용 에러 (Tool Use Errors)

**증상:** "Tool execution failed" 또는 권한 거부

**원인:**
- 종속성 누락 (npm, python 등)
- 파일 권한 부족
- 경로를 찾을 수 없음

**해결 방법:**
```bash
# 필수 도구 설치 여부 확인
which node python3 npm git

# 후크 스크립트 권한 수정
chmod +x ~/.claude/plugins/cache/*/hooks/*.sh
chmod +x ~/.claude/plugins/cache/*/skills/*/hooks/*.sh

# PATH에 필요한 바이너리가 포함되어 있는지 확인
echo $PATH
```

---

## 후크 및 워크플로우 오류

### 후크가 실행되지 않음 (Hooks Not Firing)

**증상:** Pre/post 후크가 실행되지 않음

**원인:**
- settings.json에 후크가 등록되지 않음
- 잘못된 후크 구문
- 후크 스크립트가 실행 가능하지 않음

**해결 방법:**
```bash
# 후크 등록 확인
grep -A 10 '"hooks"' ~/.claude/settings.json

# 후크 파일 존재 및 실행 가능 여부 확인
ls -la ~/.claude/plugins/cache/*/hooks/

# 수동으로 후크 테스트
bash ~/.claude/plugins/cache/*/hooks/pre-bash.sh <<< '{"command":"echo test"}'

# 후크 재등록 (플러그인 사용 시)
# Claude Code 설정에서 플러그인 비활성화 후 다시 활성화
```

### Python/Node 버전 불일치

**증상:** "python3 not found" 또는 "node: command not found"

**원인:**
- Python/Node 설치 누락
- PATH 설정되지 않음
- 잘못된 Python 버전 (Windows)

**해결 방법:**
```bash
# Python 3 설치 (누락된 경우)
# macOS: brew install python3
# Ubuntu: sudo apt install python3
# Windows: python.org에서 다운로드

# Node.js 설치 (누락된 경우)
# macOS: brew install node
# Ubuntu: sudo apt install nodejs npm
# Windows: nodejs.org에서 다운로드

# 설치 확인
python3 --version
node --version
npm --version

# Windows: python (python3 아님) 명령어가 작동하는지 확인
python --version
```

### 개발 서버 차단 오탐지 (Dev Server Blocker False Positives)

**증상:** "dev"가 포함된 정상적인 명령어임에도 후크가 차단함

**원인:**
- Heredoc 내용이 패턴 매칭을 유발
- 인자에 "dev"가 포함된 비개발 명령어

**해결 방법:**
```bash
# 이 문제는 v1.8.0 이상에서 수정되었습니다 (PR #371)
# 플러그인을 최신 버전으로 업그레이드하십시오.

# 임시 방편: 개발 서버를 tmux로 실행
tmux new-session -d -s dev "npm run dev"
tmux attach -t dev

# 필요시 후크 일시 비활성화
# ~/.claude/settings.json을 편집하여 pre-bash 후크 제거
```

---

## 설치 및 설정

### 플러그인이 로드되지 않음 (Plugin Not Loading)

**증상:** 설치 후 플러그인 기능을 사용할 수 없음

**원인:**
- 마켓플레이스 캐시가 업데이트되지 않음
- Claude Code 버전 호환성 문제
- 손상된 플러그인 파일

**해결 방법:**
```bash
# 캐시를 변경하기 전 플러그인 캐시 확인
ls -la ~/.claude/plugins/cache/

# 플러그인 캐시를 직접 삭제하는 대신 백업 후 이동
mv ~/.claude/plugins/cache ~/.claude/plugins/cache.backup.$(date +%Y%m%d-%H%M%S)
mkdir -p ~/.claude/plugins/cache

# 마켓플레이스에서 재설치
# Claude Code → Extensions → Everything Claude Code → Uninstall
# 그 후 마켓플레이스에서 다시 설치

# Claude Code 버전 확인
claude --version
# Claude Code 2.0 이상 필요

# 수동 설치 (마켓플레이스 실패 시)
git clone https://github.com/affaan-m/everything-claude-code.git
cp -r everything-claude-code ~/.claude/plugins/ecc
```

### 패키지 매니저 감지 실패

**증상:** 잘못된 패키지 매니저 사용 (pnpm 대신 npm 등)

**원인:**
- 잠금 파일(lock file)이 없음
- CLAUDE_PACKAGE_MANAGER가 설정되지 않음
- 여러 잠금 파일로 인한 감지 혼선

**해결 방법:**
```bash
# 전역적으로 선호하는 패키지 매니저 설정
export CLAUDE_PACKAGE_MANAGER=pnpm
# ~/.bashrc 또는 ~/.zshrc에 추가

# 또는 프로젝트별로 설정
echo '{"packageManager": "pnpm"}' > .claude/package-manager.json

# 또는 package.json 필드 사용
npm pkg set packageManager="pnpm@8.15.0"

# 경고: 잠금 파일을 삭제하면 설치된 종속성 버전이 변경될 수 있습니다.
# 먼저 잠금 파일을 커밋하거나 백업한 후, 새로 설치하고 CI를 다시 실행하십시오.
# 패키지 매니저를 의도적으로 변경할 때만 수행하십시오.
rm package-lock.json  # pnpm/yarn/bun을 사용하는 경우
```

---

## 성능 문제

### 응답 시간 지연 (Slow Response Times)

**증상:** 에이전트 응답에 30초 이상 소요됨

**원인:**
- 대용량 관찰 파일
- 너무 많은 활성 후크
- API 네트워크 지연

**해결 방법:**
```bash
# 대용량 관찰 내용을 삭제하는 대신 아카이브
archive_dir="$HOME/.claude/homunculus/archive/$(date +%Y%m%d)"
mkdir -p "$archive_dir"
find ~/.claude/homunculus/projects -name "observations.jsonl" -size +10M -exec sh -c '
  for file do
    base=$(basename "$(dirname "$file")")
    gzip -c "$file" > "'"$archive_dir"'/${base}-observations.jsonl.gz"
    : > "$file"
  done
' sh {} +

# 사용하지 않는 후크 일시 비활성화
# ~/.claude/settings.json 편집

# 활성 관찰 파일을 작게 유지
# 대규모 아카이브는 ~/.claude/homunculus/archive/ 아래에 보관해야 합니다.
```

### 높은 CPU 사용량

**증상:** Claude Code가 CPU를 100% 점유함

**원인:**
- 무한 관찰 루프
- 대규모 디렉토리에 대한 파일 감시(File watching)
- 후크 내 메모리 누수

**해결 방법:**
```bash
# 폭주하는 프로세스 확인
top -o cpu | grep claude

# continuous learning 일시 비활성화
touch ~/.claude/homunculus/disabled

# Claude Code 재시작
# Cmd/Ctrl+Q 후 다시 열기

# 관찰 파일 크기 확인
du -sh ~/.claude/homunculus/*/
```

---

## 일반적인 에러 메시지

### "EACCES: permission denied"

**해결 방법:**
```bash
# 후크 권한 수정
find ~/.claude/plugins -name "*.sh" -exec chmod +x {} \;

# 관찰 디렉토리 권한 수정
chmod -R u+rwX,go+rX ~/.claude/homunculus
```

### "MODULE_NOT_FOUND"

**해결 방법:**
```bash
# 플러그인 종속성 설치
cd ~/.claude/plugins/cache/everything-claude-code
npm install

# 또는 수동 설치의 경우
cd ~/.claude/plugins/ecc
npm install
```

### "spawn UNKNOWN"

**해결 방법:**
```bash
# Windows 전용: 스크립트가 올바른 줄바꿈 형식을 사용하는지 확인
# CRLF를 LF로 변환
find ~/.claude/plugins -name "*.sh" -exec dos2unix {} \;

# 또는 dos2unix 설치
# macOS: brew install dos2unix
# Ubuntu: sudo apt install dos2unix
```

---

## 도움 받기

이슈가 계속 발생한다면:

1. **GitHub Issues 확인**: [github.com/affaan-m/everything-claude-code/issues](https://github.com/affaan-m/everything-claude-code/issues)
2. **디버그 로깅 활성화**:
   ```bash
   export CLAUDE_DEBUG=1
   export CLAUDE_LOG_LEVEL=debug
   ```
3. **진단 정보 수집**:
   ```bash
   claude --version
   node --version
   python3 --version
   echo $CLAUDE_PACKAGE_MANAGER
   ls -la ~/.claude/plugins/cache/
   ```
4. **이슈 오픈**: 디버그 로그, 에러 메시지 및 진단 정보를 포함하여 이슈를 등록해 주십시오.

---

## 관련 문서 (Related Documentation)

- [README.md](./README.md) - 설치 및 기능
- [CONTRIBUTING.md](./CONTRIBUTING.md) - 개발 가이드라인
- [docs/](./docs/) - 상세 문서
- [examples/](./examples/) - 사용 예시
