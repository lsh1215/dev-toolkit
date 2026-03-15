---
name: e2e-testing
description: E2E/QA testing with agent-browser CLI for token-efficient browser automation
triggers:
  - "e2e test"
  - "e2e 테스트"
  - "browser test"
  - "브라우저 테스트"
  - "qa test"
  - "qa 테스트"
  - "페이지 테스트"
  - "화면 테스트"
  - "agent-browser"
  - "end to end"
---

## E2E Testing Skill — agent-browser CLI

Browser automation for QA/E2E testing using Vercel's `agent-browser` CLI.
Uses accessibility tree refs (`@e1`, `@e2`) instead of CSS selectors — 5.7x more token-efficient than Playwright MCP.

### Prerequisites

```bash
npm install -g agent-browser
agent-browser install  # installs Chromium
```

### Core Workflow

**1. Open & Snapshot (understand the page)**
```bash
agent-browser open <URL> && agent-browser wait --load networkidle && agent-browser snapshot -i
```
- `snapshot -i` returns interactive elements only (buttons, inputs, links) with refs like `@e1`, `@e2`
- Read the accessibility tree to understand the page structure

**2. Interact (fill forms, click buttons)**
```bash
agent-browser fill @e3 "test@example.com"
agent-browser fill @e4 "password123"
agent-browser click @e5                    # Submit button
agent-browser wait --load networkidle
```

**3. Assert (verify results)**
```bash
agent-browser snapshot -i                  # Check new page state
agent-browser get text @e10                # Get specific element text
agent-browser get url                      # Check current URL
agent-browser screenshot result.png        # Visual evidence
```

**4. Chain Commands (single bash call)**
```bash
agent-browser open localhost:3000 && \
  agent-browser wait --load networkidle && \
  agent-browser snapshot -i
```

### Command Reference

| Command | Description |
|---------|-------------|
| `open <url>` | Navigate to URL |
| `snapshot` | Full accessibility tree with refs |
| `snapshot -i` | Interactive elements only (less tokens) |
| `click @ref` | Click element by ref |
| `fill @ref "text"` | Clear and fill input |
| `type @ref "text"` | Type into element (append) |
| `press Enter` | Press keyboard key |
| `select @ref "value"` | Select dropdown option |
| `check @ref` / `uncheck @ref` | Checkbox toggle |
| `scroll down 500` | Scroll direction + pixels |
| `wait @ref` | Wait for element to appear |
| `wait 2000` | Wait milliseconds |
| `wait --load networkidle` | Wait for network idle |
| `get text @ref` | Get element text content |
| `get url` | Get current page URL |
| `get value @ref` | Get input value |
| `screenshot [path]` | Take screenshot |
| `screenshot --full` | Full page screenshot |
| `screenshot --annotate` | Labeled screenshot for AI review |
| `eval "js code"` | Execute JavaScript |
| `close` | Close browser |

### Selector Types

```bash
@e1                          # Ref from snapshot (preferred)
"button:has-text('Submit')"  # CSS + text selector
"#email"                     # CSS ID selector
".btn-primary"               # CSS class selector
```

### Testing Patterns

**Auth Flow Test:**
```bash
agent-browser open localhost:3000/auth && agent-browser wait --load networkidle && agent-browser snapshot -i
# Read refs → find email input, password input, login button
agent-browser fill @e3 "test@example.com" && agent-browser fill @e4 "password123" && agent-browser click @e5
agent-browser wait --load networkidle && agent-browser get url
# Assert: URL changed after successful login
```

**Form Submission Test:**
```bash
agent-browser open localhost:3000/contact && agent-browser wait --load networkidle && agent-browser snapshot -i
agent-browser fill @e2 "John" && agent-browser fill @e3 "john@test.com" && agent-browser fill @e4 "Hello"
agent-browser click @e5  # Submit
agent-browser wait --load networkidle && agent-browser snapshot -i
# Assert: success message visible
```

**Navigation & Content Test:**
```bash
agent-browser open localhost:3000 && agent-browser wait --load networkidle
agent-browser snapshot -i
# Assert: key navigation elements present
agent-browser click @e2  # Navigate to a page
agent-browser wait --load networkidle && agent-browser get url
# Assert: URL is correct
```

### Test Report Pattern

After each test, create a structured result:

```
## Test: [Name]
- URL: [tested URL]
- Status: PASS / FAIL
- Steps:
  1. [action] → [expected] → [actual]
  2. ...
- Screenshot: [path if captured]
- Errors: [any errors encountered]
```

### Learned Patterns

**`snapshot -i` vs full `snapshot`:**
- `snapshot -i` shows only interactive elements (buttons, links, inputs) — great for form filling and clicking
- Full `snapshot` shows ALL content including text, headings, paragraphs — use when verifying page content
- When `snapshot -i` looks unchanged after navigation, use `get url` or full `snapshot` to verify the page actually changed

**State verification via eval:**
```bash
agent-browser eval "localStorage.getItem('key')"
agent-browser eval "JSON.parse(localStorage.getItem('key')).state.field"
agent-browser eval "document.querySelector('[role=alert]')?.textContent"
```

**SPA hydration awareness:**
- After `open` (full page load), framework stores need time to rehydrate from localStorage
- Add `wait 1000` or `wait 2000` after navigation before checking auth-dependent state
- Client-side navigation (clicking links) preserves state; `open` does full page load

### Token Efficiency Rules

1. **Always use `snapshot -i`** (interactive only) instead of full `snapshot` — 60-80% less tokens
2. **Chain commands** with `&&` in a single bash call — one tool call instead of many
3. **Don't screenshot unless needed** — text assertions via `get text` are cheaper
4. **Close browser when done** — `agent-browser close`
5. **Use refs (`@e1`)** — never re-snapshot just to get CSS selectors
6. **Use `eval` for state checks** — cheaper than re-snapshotting to verify JS state

### Headless Mode (CI)

```bash
agent-browser --headless open <url> && agent-browser snapshot -i
```

### Persistent Sessions

```bash
# Save login state across runs
agent-browser --session-name myapp open localhost:3000/login
# Login via commands...
# Next time, session is restored:
agent-browser --session-name myapp open localhost:3000
```

### Connect to Existing Browser

```bash
# Connect to Chrome with remote debugging
agent-browser --cdp 9222 snapshot
# Or auto-discover running Chrome
agent-browser --auto-connect snapshot
```
