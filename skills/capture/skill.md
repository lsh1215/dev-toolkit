---
name: capture
description: Dashboard Screenshot Capture
triggers:
  - "스크린샷"
  - "screenshot"
  - "캡처"
  - "capture"
  - "화면 캡처"
  - "대시보드 캡처"
  - "dashboard capture"
  - "스크린샷 찍어"
  - "화면 찍어"
---

## Screenshot Capture Skill (Global)

외부 서비스(Grafana, AWS Console, Vercel, GitHub 등)의 화면을 캡처하여 블로그/문서용 이미지를 생성하는 skill.

### Strategy: 2-Phase Hybrid

**Phase 1 — Exploration (Playwright MCP, 최초 1회)**
selector를 모르는 경우에만 MCP로 페이지 구조 파악:
1. `browser_navigate` → URL 이동
2. `browser_snapshot` → 접근성 트리로 구조 파악 (토큰 많이 소모됨)
3. 대상 요소의 CSS selector 확정
4. 확정된 selector를 사용자에게 공유

**Phase 2 — Capture (shot-scraper CLI, 반복 사용)**
확정된 selector로 효율적 캡처:
```bash
shot-scraper "<URL>" -s "<SELECTOR>" -o <output>.png
```

selector를 이미 아는 경우 Phase 1을 건너뛰고 바로 Phase 2 실행.

### shot-scraper CLI Reference

**단일 캡처:**
```bash
# 전체 페이지
shot-scraper "<URL>" -o output.png

# CSS selector로 부분 크롭
shot-scraper "<URL>" -s "<CSS_SELECTOR>" -o output.png

# 뷰포트 지정
shot-scraper "<URL>" --width 1280 --height 800 -o output.png

# 불필요한 UI 숨기기 (사이드바, 헤더 등)
shot-scraper "<URL>" -s "<SELECTOR>" -o output.png \
  --javascript "document.querySelector('.sidebar').style.display='none'"

# Retina 해상도 (2x)
shot-scraper "<URL>" --retina -o output.png

# PDF 출력
shot-scraper pdf "<URL>" -o output.pdf
```

**인증 처리:**
```bash
# 쿠키 기반 (브라우저 인증 상태 저장)
shot-scraper auth <URL> auth.json     # 1회: 브라우저 열리면 로그인
shot-scraper -a auth.json "<URL>" -o output.png  # 이후 재사용

# Bearer 토큰 헤더
shot-scraper "<URL>" --header "Authorization: Bearer <TOKEN>" -o output.png
```

**배치 캡처 (YAML):**
```yaml
# screenshots.yml
- url: https://grafana.example.com/d/abc?kiosk
  selector: ".react-grid-item:nth-child(3)"
  output: grafana-cpu.png
  javascript: |
    document.querySelector('.sidemenu').style.display='none';

- url: https://grafana.example.com/d/abc?kiosk
  selector: ".react-grid-item:nth-child(5)"
  output: grafana-memory.png
```
```bash
shot-scraper multi screenshots.yml
```

### Output Rules

- 저장 위치: 프로젝트의 `screenshots/` 디렉토리 (없으면 생성)
- 파일명: `{서비스}-{대상}-{날짜}.png` (예: `grafana-cpu-usage-20260306.png`)
- 해상도: `--retina` 기본 사용 (블로그용 고해상도)
- 포맷: PNG 기본, 필요시 JPEG (`--quality 80`)

### Token Efficiency Rules

1. selector를 이미 아는 경우: shot-scraper CLI 직접 실행 (~500 토큰)
2. selector를 모르는 경우: Playwright MCP snapshot 최소 1회로 파악 후 CLI 전환
3. 배치 작업: 반드시 YAML + `shot-scraper multi` 사용
4. Playwright MCP의 `browser_snapshot`은 복잡한 대시보드에서 만 토큰 이상 소모 — 최소화할 것

### 캡처 실패 시 수동 캡처 안내

shot-scraper로 캡처가 안 되는 경우 (로그인 필요, Google 등 자동화 브라우저 차단 서비스) 사용자에게 아래 수동 캡처 방법을 안내한다:

> **Chrome DevTools 전체 페이지 캡처 방법:**
> 1. **F12** (또는 `Cmd + Option + I`)로 개발자 도구 열기
> 2. **`Cmd + Shift + P`** 로 Command Palette 열기
> 3. **`Capture full size screenshot`** 입력 후 선택
> 4. PNG 파일이 자동 다운로드됨 → 프로젝트의 `screenshots/` 폴더에 넣으면 됨

**참고**: Google(Gmail, Drive 등)은 보안 정책상 Playwright/Puppeteer 등 자동화 브라우저에서의 로그인을 차단하므로 `shot-scraper auth`로도 해결이 불가능하다.

### Grafana Specific Tips

- URL에 `?kiosk` 파라미터로 사이드바/헤더 자동 숨김
- `?kiosk&from=now-1h&to=now` 로 시간 범위 지정
- Service Account Token 발급 후 `--header` 로 인증
- 개별 패널 공유 URL: `?viewPanel=<panelId>&kiosk`
