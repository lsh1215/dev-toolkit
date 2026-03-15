---
name: diagram-excalidraw
description: Generate professional system architecture diagrams using Excalidraw. Expert-level system design visuals.
triggers:
  - "시스템 아키텍처"
  - "system architecture"
  - "아키텍처 다이어그램"
  - "architecture diagram"
  - "다이어그램 그려"
  - "블로그 다이어그램"
  - "excalidraw"
  - "sequence diagram 그려"
  - "시퀀스 다이어그램 그려"
  - "flow diagram"
  - "플로우 다이어그램"
  - "system design"
  - "시스템 설계"
  - "MSA"
  - "마이크로서비스"
---

## Excalidraw Professional Architecture Diagram Skill

**MANDATORY**: Use Excalidraw MCP tools exclusively. Never fall back to Mermaid, PlantUML, or ASCII art.

### Style Selection

This skill supports two visual styles. Ask the user which style to use, or infer from context:

| Style | When to use | Visual character |
|-------|-------------|-----------------|
| **Style 1: Minimal** (default) | System design interviews, whiteboard-style, blog posts | Transparent boxes, hand-drawn feel, minimal color |
| **Style 2: Icon** | Presentations, formal documentation, detailed infra diagrams | Library icons (DB cylinders, browsers, phones), richer visuals |

If the user says "아이콘", "icon", "아이콘 스타일", "스타일 2" → use Style 2.
Otherwise → default to Style 1.

**Both styles share**: Design Philosophy (Section 2), Layout System (Section 3), Arrow Rules (Section 6), Templates (Section 7), Generation Workflow (Section 8), Quality Checklist (Section 9). They differ in visual style (Section 4 vs 4B) and component rendering.

---

### 1. MCP Setup

**Canvas**: `http://localhost:3111` — 사용자가 브라우저에서 열어두면 실시간 반영.

Canvas 서버가 꺼져있으면:
```bash
docker start excalidraw-canvas
# 컨테이너 없으면:
docker run -d --name excalidraw-canvas -p 3111:3000 --restart unless-stopped ghcr.io/yctimlin/mcp_excalidraw-canvas:latest
```

**실시간 미리보기 안내 (MANDATORY)**: 다이어그램 시작 시 반드시 안내:
> 브라우저에서 http://localhost:3111 을 열어두면 실시간으로 변경사항을 확인할 수 있습니다.

**MCP 도구 (26개)**:
| 카테고리 | 도구 |
|---|---|
| 요소 CRUD | `create_element`, `get_element`, `update_element`, `delete_element`, `query_elements`, `batch_create_elements`, `duplicate_elements` |
| 레이아웃 | `align_elements`, `distribute_elements`, `group_elements`, `ungroup_elements` |
| Canvas 상태 | `describe_scene`, `get_canvas_screenshot`, `clear_canvas`, `set_viewport` |
| 저장/내보내기 | `export_scene`, `import_scene`, `export_to_image`, `snapshot_scene`, `restore_snapshot` |
| 변환 | `create_from_mermaid` |
| 가이드 | `read_diagram_guide`, `read_me` |

---

### 2. Design Philosophy

These principles are grounded in cognitive science and information design research. They are not stylistic preferences — they are how humans actually process visual information.

**Gutenberg Diagram (Edmund Arnold)**: The eye scans from top-left (Primary Optical Area) to bottom-right (Terminal Area). Place entry points (clients, users) top-left, terminal states (databases, external APIs) bottom-right. Causality and reading direction must align.

**Tufte's Data-Ink Ratio**: Every visual element must carry structural meaning. No decorative fills, no gradient backgrounds, no drop shadows. If a color doesn't encode architecture role, remove it. Transparent backgrounds by default.

**Miller's 7±2 Limit**: Working memory holds ~7 chunks. A single diagram view should contain no more than 7-9 primary nodes. For larger systems, split into multiple diagrams at different abstraction levels (C4 model).

**Gestalt Proximity**: Physical distance is the strongest grouping signal. Components that are close together are perceived as a group — more powerfully than any bounding box or color. Use whitespace intentionally to separate concerns.

**Arrow Crossing Minimization**: Edge crossings at shallow angles (<30°) cause measurable reading delays (Ware et al., 2008). Eliminate crossings by reordering components. If unavoidable, ensure crossings are near 90°.

**3-Color Rule**: The visual system reliably distinguishes ~6-8 hues. Limit to: neutral base (near-black #1e1e1e) + structural accent (system boundary color) + one semantic color (e.g., purple for async). More colors = more noise.

**Fowler's Principle**: "Comprehensiveness is the enemy of comprehensibility." A diagram showing the right things simply outperforms a diagram showing everything beautifully.

---

### 3. Layout System

**Primary flow: Left-to-Right** (following Gutenberg Diagram for LTR readers).

All coordinates and dimensions MUST be multiples of 20 (grid-snap).

**Column Structure (L→R)**:
```
Col 1  x=80    External actors (Client, Users) — outside system boundary
Col 2  x=380   Entry point (API Gateway, Load Balancer)
Col 3  x=700   Services (stacked vertically, 180px row gap)
Col 4  x=1060  Data stores (same row as their service — horizontal alignment)
Col 5  x=1380  External dependencies (Stripe, third-party APIs) — outside boundary
```

**Vertical row spacing**: 180px between rows.

**Horizontal centering for vertical stacks**: When stacking N components of width W with gap G:
- Total height = N × H + (N−1) × G
- Start Y = center_y − (Total height / 2)

**Critical layout rule — Same-Row Alignment**:
Service and its dedicated DB MUST be on the same horizontal row. This ensures Service→DB arrows are perfectly horizontal, eliminating all vertical crossing issues.

```
Row 1:  Product Service  ──→  Products DB
Row 2:  User Service     ──→  Users DB
Row 3:  Order Service    ──→  Orders DB
Row 4:  Payment Service  ──→  Payments DB  ──→  Stripe API
```

**Row ordering strategy**: Order rows by user journey flow (top=first interaction, bottom=last). This naturally places event publishers (Order, Payment) near the bottom, close to the Event Bus below.

**Skip empty layers**. Never leave empty vertical space.

**System boundary**: A large rectangle with `strokeWidth: 4` and accent color (e.g., `#f08c00` orange) enclosing all internal components. External actors (Client, third-party APIs) stay OUTSIDE the boundary.

**Logical grouping**: Use `strokeStyle: "dashed"` rectangles with `strokeWidth: 2` inside the system boundary to cluster related services (e.g., "Microservices" cluster).

---

### 4. Visual Style

**Global defaults for ALL components**:
```json
{
  "roughness": 1,
  "strokeWidth": 2,
  "strokeColor": "#1e1e1e",
  "backgroundColor": "transparent",
  "fontFamily": 5,
  "fontSize": 20
}
```

**Why these values**:
- `roughness: 1` — Slight hand-drawn feel. Signals "this is a design sketch, not a blueprint" (Fowler's sketching mode). Invites discussion and revision.
- `backgroundColor: transparent` — Tufte's data-ink ratio. No decorative fills. Structure is communicated through borders, position, and grouping.
- `fontFamily: 5` (Excalifont) — Clean, consistent, slightly informal.
- `fontSize: 20` — Uniform size for all component labels. Hierarchy is through position and nesting, not font size.
- `strokeColor: "#1e1e1e"` (near-black) — One neutral color for all components. Color is reserved for structural boundaries only.

**Stroke width hierarchy** (communicates architectural importance):
| strokeWidth | Purpose |
|-------------|---------|
| 4 | System boundary (outermost container) |
| 2 | Components, arrows, logical groupings |
| 1 | Icon internals (if using custom shapes) |

**Color usage** (3-color rule):
| Color | Hex | Purpose |
|-------|-----|---------|
| Near-black | #1e1e1e | ALL components, ALL arrows (default) |
| Orange | #f08c00 | System boundary rectangle (strokeColor) |
| Purple | #6741d9 | Async/event arrows and labels ONLY |

The system boundary gets white fill (`#ffffff`) to visually separate the system interior from the canvas background.

External systems: `strokeStyle: "dashed"` (signals "outside our control").
Logical clusters: `strokeStyle: "dashed"`, `strokeColor: "#1e1e1e"`.

---

### 5. Component Naming

**Text pattern**: Short, descriptive name only.
- Services: `"Order Service"`, `"User Service"` (NOT `"Order Service\n[Spring Boot]"`)
- Databases: `"Orders DB"`, `"Users DB"`
- Infrastructure: `"Event Bus (Kafka)"`, `"API Gateway"`
- External: `"Stripe API"`

**Technology annotations**: Place as a SEPARATE small text element at the **top-left corner** of the component box, NOT inside the box or on arrows.
```json
{ "type": "text", "x": box_x + 2, "y": box_y - 16, "text": "PostgreSQL", "fontSize": 12, "fontFamily": 5, "strokeColor": "#868e96" }
```
This keeps the component box clean while still showing the technology stack. Only annotate where the technology choice is architecturally significant (e.g., MongoDB vs PostgreSQL matters; "Spring Boot" on every service does not).

---

### 4B. Visual Style — Style 2: Icon Mode

Use this style when the user requests icons, richer visuals, or presentation-quality output.

**Library location**: `~/.claude/skills/diagram-excalidraw/libraries/`

| File | Items | 용도 | 사용 시점 |
|------|-------|------|----------|
| `general-icons.excalidrawlib` | 36 | drwnio + technology-logos 통합. 범용 아키텍처 아이콘 + 기술 로고 | **기본 (Style 2 default)** |
| `aws-icons.excalidrawlib` | 249 | AWS 서비스 공식 아이콘 | "AWS 아키텍처" 요청 시 |
| `gcp-icons.excalidrawlib` | 83 | GCP 서비스 아이콘 | "GCP 아키텍처" 요청 시 |

**아이콘 선택 우선순위**:
1. **기본**: `general-icons.excalidrawlib`
2. **"AWS로 그려줘"**: `aws-icons.excalidrawlib`
3. **"GCP로 그려줘"**: `gcp-icons.excalidrawlib`

**general-icons.excalidrawlib (기본 아이콘)**:

**구조**: it-logos (index 0-30, 우선순위 1, 이름 있음) + drwnio filtered (index 31-45, fallback, 중복 제거됨)

it-logos (index 0-30) — **우선 사용** (이름 기반 검색):
| Index | Name | 용도 |
|-------|------|------|
| 4 | Docker | 컨테이너 |
| 7 | Firebase | 알림/푸시 |
| 9 | GitLab | Git |
| 10 | **Kafka** | Event Bus / Message Queue |
| 12 | **Kubernetes** | 오케스트레이션 |
| 16 | Next | Next.js 프론트엔드 |
| 18 | **Python** | Python 서비스 |
| 19 | **React** | Web Client |
| 24 | Svelte | Svelte 프론트엔드 |
| 28 | Vercel | 배포 |
| 29 | Vue | Vue 프론트엔드 |

drwnio (index 31-45) — **fallback** (범용 아키텍처 아이콘):
| Index | Name | 용도 |
|-------|------|------|
| 31 | Server | 서버 |
| 32 | **DB Cylinder** | 데이터베이스 |
| 33 | JSON File | 설정/파일 |
| 34 | **Redis** | 캐시 |
| 35 | Node.js | Node 서비스 |
| 36 | Service Icon | 범용 서비스 |
| 37 | Network | 네트워크 |
| 38 | Server Alt | 서버 대체 |
| 39 | **Globe/Internet** | Web Client / 인터넷 |
| 40 | Storage | 스토리지 |
| 41 | Container | 컨테이너 |
| 42 | **User** | 사용자/모바일 |
| 43 | API | API 엔드포인트 |
| 44 | Large Service | 큰 서비스 |
| 45 | DNS | DNS |

**How to place library icons**:

각 아이콘을 **개별 .excalidraw 파일**로 저장 후 `import_scene`으로 하나씩 import.

```python
import json, uuid, copy

LIB_DIR = '~/.claude/skills/diagram-excalidraw/libraries'

def save_icon(lib_path, index, target_x, target_y, prefix, out_path):
    """Extract one library item, assign unique IDs, save as .excalidraw"""
    with open(lib_path) as f:
        lib = json.load(f)
    item = copy.deepcopy(lib['library'][index])
    min_x = min(e['x'] for e in item)
    min_y = min(e['y'] for e in item)
    grp = f"{prefix}-grp"
    for e in item:
        e['x'] = e['x'] - min_x + target_x
        e['y'] = e['y'] - min_y + target_y
        e['id'] = f"{prefix}-{uuid.uuid4().hex[:8]}"
        if e.get('groupIds'):
            e['groupIds'] = [grp]
    scene = json.dumps({'type': 'excalidraw', 'version': 2, 'elements': item})
    with open(out_path, 'w') as f:
        f.write(scene)
```

Then import each file individually:
```
import_scene(filePath="icon-db1.excalidraw", mode="merge")
```

**CRITICAL constraints (에러 방지)**:
- **One icon per file**: 여러 아이콘을 한 .excalidraw 파일에 합치면 import 시 누락됨. 반드시 아이콘 1개 = 파일 1개.
- **Unique IDs**: 같은 라이브러리 아이템을 여러 번 복제할 때 반드시 `uuid.uuid4()`로 각 요소 ID + groupId 유니크화. 중복 ID는 마지막 하나만 렌더링됨.
- **System boundary**: Style 2에서는 `backgroundColor: "transparent"` 필수. white fill은 아이콘을 z-order로 가림.
- **filePath 제한**: `import_scene`의 `filePath`는 프로젝트 디렉토리 내부만 가능. `/tmp/` 등 외부 경로 차단됨. 반드시 프로젝트 루트에 temp 파일 생성.
- **temp 파일 정리**: import 성공 후 즉시 `.excalidraw` temp 파일 삭제 (`rm`).
- **scale 처리**: 아이콘 원본이 300-600px로 크므로 scale 팩터(0.15~0.4)를 적용. width, height, points 배열 모두 scale 적용 필수.
- **import 순서**: 먼저 `batch_create_elements`로 모든 rectangle/arrow/text 생성 → 그 다음 `import_scene`으로 아이콘 추가 (z-order: 아이콘이 위에 렌더링됨)

**Style 2 component rendering** (기본: general-icons):
- 아이콘이 컴포넌트 의미와 **정확히 맞을 때만** 사용. 억지로 넣지 않는다.
- 맞지 않으면 Style 1과 동일한 rectangle + 텍스트로 대체.
- DB → index 32 (DB Cylinder, drwnio) + 아래 텍스트 레이블
- Kafka/Queue → index 10 (Kafka, it-logos) + 아래 텍스트
- Kubernetes → index 12 (Kubernetes, it-logos)
- Docker → index 4 (Docker, it-logos)
- Redis/Cache → index 34 (Redis, drwnio)
- Python Service → index 18 (Python, it-logos)
- React Client → index 19 (React, it-logos) — 단, 어울리지 않으면 rectangle
- Client (Web/Mobile), API Gateway, 일반 Service → rectangle 유지 (아이콘 불필요)
- **원칙: 아이콘은 기술 스택을 시각적으로 즉시 인식 가능할 때만 사용. 모호하면 박스가 낫다.**

**Arrow binding for icons**: 아이콘은 line/ellipse/draw 요소로 구성 — arrow binding 불가. 같은 위치에 `opacity: 0` 투명 rectangle을 생성하고 화살표를 그 rectangle에 바인딩.

**Style 2 inherits from Style 1**: Layout system, arrow rules, design philosophy, templates, quality checklist 모두 동일. 컴포넌트 렌더링만 아이콘으로 교체.

**AWS Icons 주요 인덱스** (aws-icons.excalidrawlib, "AWS로 그려줘" 요청 시):

| Index | Name | Use for |
|-------|------|---------|
| 52 | Aurora | Aurora DB |
| 54 | DynamoDB | NoSQL |
| 62 | RDS | SQL DB |
| 65 | ElastiCache | Cache |
| 74 | S3 | Storage |
| 78 | EC2 | Compute |
| 87 | Lambda | Serverless |
| 94 | EKS | K8s |
| 98 | ECS | Container |
| 104 | Fargate | Serverless Container |
| 105 | SNS | Pub/Sub |
| 106 | SQS | Queue |
| 110 | EventBridge | Event Bus |
| 209 | CloudFront | CDN |
| 228 | ELB | Load Balancer |
| 233 | API Gateway | API GW |

---

### 6. Arrow & Connection Rules

**Global arrow defaults**:
```json
{
  "strokeWidth": 2,
  "strokeColor": "#1e1e1e",
  "roughness": 1,
  "endArrowhead": "arrow"
}
```

**CRITICAL**: Always use `startElementId` and `endElementId` for arrow binding. Never use manual coordinates.

**Arrow text: Use floating text labels, NOT built-in arrow `text` property.**

Built-in arrow labels auto-place at the midpoint and CANNOT be repositioned, causing overlap when arrows cross other elements. Instead, create a separate `type: "text"` element positioned near the arrow's source or along the arrow path.

```json
// WRONG — built-in label, causes overlap
{ "type": "arrow", ..., "text": "REST" }

// CORRECT — floating text near the arrow
{ "type": "text", "x": 610, "y": 266, "text": "REST", "fontSize": 16, "fontFamily": 5, "strokeColor": "#868e96" }
```

**Label placement strategy**:
- Protocol labels (HTTPS, REST, gRPC): Place as floating text between source and target, slightly above the arrow path
- One label per arrow group is sufficient (e.g., one "REST" label for 4 Gateway→Service arrows, not 4 separate labels)
- Async labels: Place near the Event Bus with the async accent color

**Connection types**:
| Type | strokeStyle | strokeColor | Label example |
|------|-------------|-------------|--------------|
| Sync call | solid | #1e1e1e | "HTTPS", "REST", "gRPC" |
| Async event | dashed | #6741d9 | "Pub/Sub", "Event" |
| Data access | solid | #1e1e1e | (usually unlabeled — the DB box name is sufficient) |
| External API | solid | #1e1e1e | "REST API" |

**When NOT to label arrows**:
- Service → its own DB: The horizontal arrow's meaning is obvious from context. No label needed.
- Multiple identical connections: Use one shared label (e.g., one "REST" for all Gateway→Service arrows).

---

### 7. Diagram Templates

**Template selection logic**:
- "시스템 설계", "system design" → **System Design Template**
- "MSA", "마이크로서비스", "microservice" → **MSA Template**
- "파이프라인", "data flow", "ETL" → **Data Pipeline Template**
- Default → System Design Template

#### System Design Template
Left-to-right flow:
```
Client → LB/CDN → API Gateway → Services → Cache/DB
```
- Entry point (Client) at top-left, outside system boundary
- Services stacked vertically in center column
- Data stores to the right, same row as their service

#### MSA Template
Left-to-right flow with per-service databases:
```
Client → API Gateway → [Services column] → [DB column]
                              ↓
                        Event Bus → Notification
```
- System boundary (orange, strokeWidth 4) encloses Gateway through DBs
- Dashed cluster around Microservices column
- Each Service → DB on same horizontal row (zero crossing)
- Row order follows user journey (browse → auth → order → pay)
- Event Bus below services; publishers (Order, Payment) at bottom rows, closest to Event Bus
- Async arrows in purple dashed (#6741d9)
- External systems (Stripe) outside boundary, connected via dashed-border box

#### Data Pipeline Template
Left-to-right horizontal flow:
```
Sources → Ingestion → Processing → Storage → Serving → Consumers
```
- Stages as columns, components stacked vertically within each stage
- Stage gap: 240px horizontal

---

### 8. Generation Workflow

Execute in this exact order:

```
Step 1: clear_canvas
Step 2: Plan layout — calculate all positions using Column Structure + Same-Row Alignment
Step 3: batch_create_elements — System boundary + logical grouping zones
Step 4: batch_create_elements — All components (assign custom `id` to each)
Step 5: batch_create_elements — All arrows (using startElementId/endElementId) + floating text labels + title
Step 6: get_canvas_screenshot — Visual verification
Step 7: Quality Checklist review — fix issues with update_element / delete_element
```

**ID convention**: Use descriptive, kebab-case IDs:
- `"client"`, `"gw"`, `"svc-order"`, `"svc-user"`, `"db-orders"`, `"event-bus"`, `"stripe"`

**Title placement**: Top-left corner (Gutenberg primary optical area), `fontSize: 28`, `fontFamily: 5`.

---

### 9. Quality Checklist

After `get_canvas_screenshot`, verify:

- [ ] L→R flow: Entry points left, data stores right (Gutenberg)
- [ ] All components grid-aligned, no overlaps
- [ ] Same-category components have identical dimensions
- [ ] Service and its DB are on the same horizontal row
- [ ] Zero or near-zero arrow crossings
- [ ] Sync arrows solid, async arrows dashed + purple
- [ ] No built-in arrow text labels (use floating text instead)
- [ ] Technology annotations at top-left of boxes (not on arrows)
- [ ] Title at top-left
- [ ] System boundary clearly separates internal from external
- [ ] Generous whitespace between groups (Gestalt proximity)
- [ ] ≤ 9 primary nodes per view (Miller's limit)
- [ ] All arrows use startElementId/endElementId binding

If any item fails, fix with `update_element` or `delete_element`. Do NOT regenerate the entire diagram.

---

### 10. Saving & Export

- **File save**: `export_scene` → `.excalidraw` JSON file
- **Image export**: `export_to_image` → PNG/SVG
- **Snapshot**: `snapshot_scene` / `restore_snapshot` → backup/restore during work

### 11. Fallback

If MCP server is unavailable:
1. Generate `.excalidraw` JSON file directly for opening in excalidraw.com
2. Guide user to start the MCP server

### 12. Exception

If the user explicitly asks for a README or markdown-embedded diagram, Mermaid is acceptable.
For any blog, presentation, or standalone diagram, Excalidraw is mandatory.
