# 클로드 코드 훅 (Hooks)

훅은 Claude Code 도구가 실행되기 전후에 트리거되는 이벤트 기반 자동화 프로그램입니다. 코드 품질 강제, 에러 조기 발견, 반복적인 체크 작업 자동화 등을 위해 사용됩니다.

## 동작 원리

```text
사용자 요청 → Claude 도구 선택 → PreToolUse 훅 실행 → 도구 실행 → PostToolUse 훅 실행
```

* **PreToolUse**: 도구 실행 직전에 작동합니다. 실행을 **차단**(종료 코드 2)하거나 **경고**(stderr 출력 후 계속 진행)를 보낼 수 있습니다.
* **PostToolUse**: 도구 실행 완료 후에 작동합니다. 출력값을 분석할 수 있지만 실행을 되돌릴 수는 없습니다.
* **Stop**: Claude의 각 응답이 끝날 때마다 작동합니다.
* **SessionStart / SessionEnd**: 세션의 시작과 종료 시점에 작동합니다.
* **PreCompact**: 컨텍스트 압축(Compact) 직전에 상태 저장을 위해 작동합니다.

---

## 주요 훅 목록

### PreToolUse 훅 (사전 점검)

| 훅 명칭 | 대상 도구 | 주요 동작 및 특징 | 결과 |
|---------|-----------|-------------------|------|
| **개발 서버 차단** | `Bash` | tmux 외부에서 `npm run dev` 등 실행 시 차단 (로그 접근성 보장) | 차단 (코드 2) |
| **Tmux 권장** | `Bash` | 오래 걸리는 명령어(테스트, 빌드 등) 실행 시 tmux 사용 권장 | 경고 (코드 0) |
| **Git Push 확인** | `Bash` | `git push` 전 변경 사항 최종 검토 요청 | 경고 (코드 0) |
| **문서 파일 경고** | `Write` | 비표준 폴더에 `.md` 생성 시 권장 경로(docs/, skills/ 등) 안내 | 경고 (코드 0) |
| **전략적 압축** | `Edit/Write` | 약 50회 도구 호출마다 `/compact` 실행 권장 | 경고 (코드 0) |

### PostToolUse 훅 (사후 처리)

| 훅 명칭 | 대상 도구 | 주요 동작 및 특징 |
|---------|-----------|-------------------|
| **PR 기록기** | `Bash` | `gh pr create` 성공 후 PR URL 및 리뷰 명령어 기록 |
| **품질 게이트** | `Edit/Write` | 코드 수정 후 즉각적인 품질 검사(Linter 등) 실행 |
| **Prettier 포맷터** | `Edit` | JS/TS 파일 수정 후 자동으로 코드 스타일 정리 |
| **TypeScript 체크** | `Edit` | `.ts`/`.tsx` 수정 후 `tsc --noEmit`으로 타입 에러 검증 |
| **Console.log 경고** | `Edit` | 수정된 파일에 `console.log`가 남아있는 경우 경고 |

### 세션 및 라이프사이클 훅

* **SessionStart**: 이전 컨텍스트 로드 및 패키지 매니저 자동 감지
* **Stop**: 매 응답 후 `console.log` 잔류 여부 재검사, 세션 상태 요약 및 학습 패턴 추출
* **Cost Tracker**: 실행 비용(토큰 사용량 등)에 대한 경량 트래킹 태그 생성

---

## 훅 커스터마이징

### 훅 비활성화
`hooks.json`에서 해당 항목을 삭제하거나, `~/.claude/settings.json`에서 특정 도구의 훅 목록을 비워 덮어쓰기 하십시오.

### 실행 프로필 제어 (환경 변수)
파일 수정 없이 환경 변수로 훅의 수준을 조절할 수 있습니다.
* `export ECC_HOOK_PROFILE=standard` (minimal, standard, strict 중 선택)
* `export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder"` (특정 ID 비활성화)

---

## 나만의 훅 만들기

훅은 표준 입력(stdin)으로 JSON 형태의 도구 입력을 받고, 표준 출력(stdout)으로 다시 JSON을 출력해야 하는 셸 명령어입니다.

**기본 구조 (Node.js 예시):**
```javascript
let data = '';
process.stdin.on('data', chunk => data += chunk);
process.stdin.on('end', () => {
  const input = JSON.parse(data);
  const toolName = input.tool_name; // "Edit", "Bash" 등

  // 경고 메시지 출력 (차단하지 않음)
  console.error('[Hook] 사용자에게 보여줄 경고 메시지');

  // 차단 시 (PreToolUse 전용)
  // process.exit(2);

  // 반드시 원본 데이터를 stdout으로 출력해야 함
  console.log(data);
});
```

**종료 코드 가이드:**
* `0`: 성공 (다음 단계 진행)
* `2`: 도구 실행 차단 (PreToolUse 단계에서만 유효)
* 기타: 에러 발생 (로그는 남기되 차단하지 않음)

**핵심**: 훅은 프로젝트의 품질 기준을 자동화하고, 사용자가 실수하기 쉬운 부분을 기계적으로 보완하여 개발 안정성을 높이는 강력한 도구입니다.
