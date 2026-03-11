# 체크포인트 명령어 (Checkpoint Command)

워크플로우에서 체크포인트를 생성하거나 검증합니다.

## 사용법

`/checkpoint [create|verify|list] [이름]`

## 체크포인트 생성 (Create Checkpoint)

체크포인트 생성 시:

1. `/verify quick`을 실행하여 현재 상태가 깨끗한지 확인합니다.
2. 체크포인트 이름으로 git stash 또는 커밋을 생성합니다.
3. `.claude/checkpoints.log`에 체크포인트를 기록합니다:

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. 체크포인트 생성 완료를 보고합니다.

## 체크포인트 검증 (Verify Checkpoint)

체크포인트와 대조하여 검증 시:

1. 로그에서 체크포인트를 읽습니다.
2. 현재 상태를 체크포인트와 비교합니다:
   - 체크포인트 이후 추가된 파일
   - 체크포인트 이후 수정된 파일
   - 현재 vs 당시의 테스트 합격률
   - 현재 vs 당시의 커버리지

3. 보고 양식:
```
체크포인트 비교: $NAME
============================
변경 파일 수: X
테스트: +Y 합격 / -Z 실패
커버리지: +X% / -Y%
빌드: [성공/실패]
```

## 체크포인트 목록 (List Checkpoints)

다음을 포함하여 모든 체크포인트를 표시합니다:
- 이름
- 타임스탬프
- Git SHA
- 상태 (현재 위치, 이전 위치, 이후 위치)

## 워크플로우 (Workflow)

일반적인 체크포인트 흐름:

```
[시작] --> /checkpoint create "기능-시작"
   |
[구현] --> /checkpoint create "핵심-완료"
   |
[테스트] --> /checkpoint verify "핵심-완료"
   |
[리팩토링] --> /checkpoint create "리팩토링-완료"
   |
[PR] --> /checkpoint verify "기능-시작"
```

## 인자 (Arguments)

$인자:
- `create <이름>` - 명명된 체크포인트 생성
- `verify <이름>` - 명명된 체크포인트와 비교 검증
- `list` - 모든 체크포인트 표시
- `clear` - 오래된 체크포인트 삭제 (최근 5개 유지)
