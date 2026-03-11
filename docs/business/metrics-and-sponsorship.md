# 지표 및 후원 플레이북 (Metrics and Sponsorship Playbook)

이 파일은 후원자 미팅 및 에코시스템 파트너 리뷰를 위한 실질적인 스크립트입니다.

## 추적해야 할 사항

모든 업데이트에서 다음 네 가지 카테고리를 사용하십시오:

1. **배포 (Distribution)** — npm 패키지 및 GitHub 앱 설치 수
2. **도입 (Adoption)** — 별(stars), 포크(forks), 기여자 수, 릴리스 주기
3. **제품 범위 (Product surface)** — 명령어/스킬/에이전트 수 및 크로스 플랫폼 지원 현황
4. **신뢰성 (Reliability)** — 테스트 통과 횟수 및 운영상 버그 해결 소요 시간

## 실시간 지표 추출

### npm 다운로드 수

```bash
# 주간 다운로드 수
curl -s https://api.npmjs.org/downloads/point/last-week/ecc-universal
curl -s https://api.npmjs.org/downloads/point/last-week/ecc-agentshield

# 최근 30일
curl -s https://api.npmjs.org/downloads/point/last-month/ecc-universal
curl -s https://api.npmjs.org/downloads/point/last-month/ecc-agentshield
```

### GitHub 저장소 도입 현황

```bash
gh api repos/affaan-m/everything-claude-code \
  --jq '{stars:.stargazers_count,forks:.forks_count,contributors_url:.contributors_url,open_issues:.open_issues_count}'
```

### GitHub 트래픽 (메인테이너 권한 필요)

```bash
gh api repos/affaan-m/everything-claude-code/traffic/views
gh api repos/affaan-m/everything-claude-code/traffic/clones
```

### GitHub 앱 설치 수

GitHub 앱 설치 수는 현재 Marketplace/앱 대시보드에서 확인하는 것이 가장 정확합니다. 다음의 최신 값을 사용하십시오:

- [ECC 도구 마켓플레이스](https://github.com/marketplace/ecc-tools)

## 현재 공개적으로 측정 불가능한 항목

- Claude 플러그인 설치/다운로드 횟수는 현재 공개 API를 통해 제공되지 않습니다.
- 파트너 협의 시에는 npm 지표 + GitHub 앱 설치 수 + 저장소 트래픽을 합산한 지표 모델을 사용하십시오.

## 제안할 후원 패키지

협상 시 다음 항목들을 시작점으로 활용하십시오:

- **파일럿 파트너 (Pilot Partner):** `월 $200`
  - 초기 파트너십 검증 및 간단한 월간 후원자 업데이트에 적합합니다.
- **성장 파트너 (Growth Partner):** `월 $500`
  - 로드맵 점검 및 구현 피드백 루프가 포함됩니다.
- **전략적 파트너 (Strategic Partner):** `월 $1,000+`
  - 다각적인 협업, 출시 지원 및 깊이 있는 운영 조율이 포함됩니다.

## 60초 요약 발표문 (Talking Track)

미팅이나 통화 시 다음 내용을 활용하십시오:

> ECC는 단순한 설정 저장소가 아닌, 에이전트 하네스 성능 시스템으로 자리매김했습니다.  
> 저희는 npm 배포, GitHub 앱 설치, 저장소 성장을 통해 도입 현황을 추적합니다.  
> Claude 플러그인 설치 수는 공개적으로 실제보다 적게 집계되는 구조이므로, 혼합 지표 모델을 사용하고 있습니다.  
> 이 프로젝트는 Claude Code, Cursor, OpenCode 및 Codex 앱/CLI를 지원하며, 운영 수준의 후크 신뢰성과 대규모의 테스트 통과 스위트를 갖추고 있습니다.

출시용 소셜 미디어 문구 스니펫은 [`social-launch-copy.md`](./social-launch-copy.md)를 참조하십시오.
    
