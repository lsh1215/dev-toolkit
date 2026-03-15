# dev-toolkit

Claude Code에서 사용할 수 있는 커스텀 스킬 모음.

## 사용법

각 스킬은 해당 폴더를 `~/.claude/skills/` 디렉토리에 복사하여 설치합니다.

Claude Code 스킬은 trigger 단어가 포함된 사용자 메시지에 자동으로 활성화됩니다. 예를 들어, "스크린샷 찍어줄래?" 메시지는 `capture` 스킬의 트리거 단어 "스크린샷"을 포함하므로 자동으로 해당 스킬이 실행됩니다.

### 설치 방법

```bash
# 모든 스킬 설치
cp -r skills/* ~/.claude/skills/

# 또는 특정 스킬만 설치
cp -r skills/capture ~/.claude/skills/
```

## 스킬 목록

| 스킬 | 설명 | 트리거 예시 | 경로 |
|------|------|-----------|------|
| **capture** | 웹 대시보드 스크린샷 캡처 | "스크린샷", "화면 캡처" | `skills/capture/skill.md` |
| **blog** | 기술 블로그 작성 | "blog", "블로그" | `skills/blog/SKILL.md` |
| **e2e-testing** | E2E/QA 테스트 (agent-browser) | "e2e 테스트", "브라우저 테스트" | `skills/e2e-testing/skill.md` |
| **diagram-excalidraw** | Excalidraw 아키텍처 다이어그램 | "아키텍처 다이어그램", "excalidraw" | `skills/diagram-excalidraw/SKILL.md` |
| **diagram-infra** | 인프라 구성도 (Python Diagrams) | "인프라 다이어그램", "배포 구성도" | `skills/diagram-infra/SKILL.md` |

## 스킬 상세 설명

### 1. capture - Dashboard Screenshot Capture

외부 서비스(Grafana, AWS Console, Vercel, GitHub 등)의 화면을 캡처하여 블로그/문서용 이미지를 생성한다. shot-scraper CLI + Playwright MCP 2-Phase Hybrid 전략으로 토큰 효율성을 극대화한다.

- 전체 페이지 또는 CSS selector로 특정 요소만 선택적 캡처
- 배치 캡처 (YAML), 인증 처리 (쿠키, Bearer 토큰), Retina 고해상도

### 2. blog - Technical Blog Writing

Orwell/Zinsser/Graham 철학 기반 기술 블로그 작성 스킬. Toulmin 논증 모델과 Steel Man 반론 구조를 적용한다.

- 3가지 템플릿: Problem-Solution (PAS), Development Journal, General Article
- 4단계 워크플로우: 템플릿 선택 → 골격 작성 → 초고 작성 → 셀프 리뷰
- Notion 발행 연동 지원

### 3. e2e-testing - E2E/QA Testing with agent-browser

Vercel의 agent-browser CLI를 사용한 토큰 효율적 브라우저 자동화 및 E2E 테스트. 접근성 트리 기반 요소 참조(`@e1`, `@e2`)로 CSS selector를 대체하여 Playwright MCP 대비 5.7배 토큰 효율적이다.

- 폼 제출, 인증 흐름, 페이지 콘텐츠 검증
- 명령 체이닝, Headless 모드, 세션 유지 지원

### 4. diagram-excalidraw - Excalidraw Architecture Diagrams

Excalidraw MCP를 사용한 전문적 시스템 아키텍처 다이어그램 생성. 인지과학 기반 디자인 원칙(Gutenberg, Tufte, Miller's 7+/-2)을 적용한다.

- 2가지 스타일: Minimal (hand-drawn), Icon (라이브러리 아이콘)
- 3가지 템플릿: System Design, MSA, Data Pipeline
- AWS/GCP 아이콘 라이브러리 지원

### 5. diagram-infra - Infrastructure Diagrams (Python Diagrams)

Python diagrams 라이브러리(mingrammer/diagrams)를 사용한 인프라 구성도 생성. Graphviz 기반 자동 레이아웃으로 코드만으로 다이어그램을 생성한다.

- Cluster, Edge를 활용한 계층적 구조 표현
- PNG/SVG 출력

## 개발 환경 설정

각 스킬의 선행 요구사항:

### capture
```bash
pip install shot-scraper
```

### e2e-testing
```bash
npm install -g agent-browser
agent-browser install
```

### diagram-excalidraw
Excalidraw MCP 서버 필요:
```bash
docker run -d --name excalidraw-canvas -p 3111:3000 --restart unless-stopped ghcr.io/yctimlin/mcp_excalidraw-canvas:latest
```

### diagram-infra
```bash
pip install diagrams
brew install graphviz  # macOS
```

## 기여 가이드

새로운 스킬을 추가할 때는 다음 구조를 따릅니다:

```
skills/
├── your-skill/
│   ├── SKILL.md (또는 skill.md)
│   ├── examples/
│   │   └── example.md
│   └── README.md (선택사항)
```

**SKILL.md 구성:**
1. YAML 헤더 (name, description, triggers)
2. 개요 및 전략
3. 설정 및 선행 요구사항
4. 주요 워크플로우
5. 명령 레퍼런스
6. 사용 패턴 및 예제
7. 토큰 효율성 규칙

## 라이선스

MIT License
