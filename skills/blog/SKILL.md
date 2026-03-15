---
description: Technical blog writing session. Writes clear and honest technical content applying Orwell/Zinsser/Graham philosophy, Toulmin argumentation, and Steel Man rebuttal. Use this skill when the user mentions blog, writing, or technical articles.
argument-hint: "[topic]" or "review"
user-invocable: true
---

# Blog - Technical Blog Writing Skill

Argument: $ARGUMENTS

---

## Writing Philosophy

The writing in this skill is grounded in the following principles. Apply them at every stage.

### 1. Writing is the process of completing thought

> "Writing about something, even something you know well, usually reveals that you didn't know it as well as you thought." — Paul Graham

It is natural for a first draft to be a mess. The thinking is not yet complete.
Writing is how thought gets organized. Therefore, the correct order is not
"understand perfectly, then write" — it is "write to complete the understanding."

### 2. Clarity is an ethical choice

> "The great enemy of clear language is insincerity." — George Orwell

Writing vaguely is deceiving the reader.
In technical writing, phrases like "handled appropriately" or "works efficiently" say nothing.
Write concretely. Write in measurable terms.

### 3. Simplicity is not simple thinking

> "The secret of good writing is to strip every sentence to its cleanest components." — William Zinsser

Writing complex ideas simply is the hardest thing of all.
Stripping clutter does not mean reducing content — it means making the content more visible.

### 4. Intellectual honesty creates persuasion

> "A theory that is not refutable by any conceivable event is non-scientific." — Karl Popper

Writing that exposes the weaknesses of its own argument earns more trust.
"This is the best approach" is weaker than "I chose this approach because of X. That said, in Y situations, Z is more appropriate." That is the language of an expert.

---

## Style Rules

### Plain sentences

- **Avoid figurative language.** Use concrete examples and data instead of metaphors.
  - ❌ "Kafka is the highway of data"
  - ✅ "Kafka stores and delivers messages in order between producers and consumers"
- **Question every modifier.** Ask every adjective and adverb: "Why are you here?"
  - ❌ "Processed very quickly"
  - ✅ "Processed within 47ms at the P99 latency threshold"
- **Write in active voice.** Do not hide the actor.
  - ❌ "The configuration was changed"
  - ✅ "The producer changes the acks setting to all"
- **Never use a long word where a short one will do.** (Orwell's Rule 2)

### Handling technical terms

- **Use technical terms only when necessary.** Replace with plain language when possible.
- **Write core technical terms in their original English.**
  - Example: "Kafka's consumer group is a logical grouping of consumers that subscribe to the same topic."
  - Only core concept words like consumer group and topic are written in English. Not every word.
- **Introduce a term with an explanation on first use, then use it without explanation thereafter.**
  - Example: "ISR (In-Sync Replica, a replica that has fully synchronized with the leader) is ..." → After that: "ISR is ..."

### Sentence rhythm

- **Do not repeat sentences of the same length.** Mix short and long sentences.
- **State core claims in short sentences.** Surrounding context can be longer.
- **Each paragraph contains only one idea.** Two to four sentences is the right amount.

---

## Argumentation Structure

### Applying the Toulmin Model

Apply this structure consciously to every technical claim:

```
Claim:    "The Kafka producer's acks must be set to all."
Grounds:  "With acks=1, data loss occurs when the leader fails."
Warrant:  "In the financial domain, message loss equals asset loss."
Qualifier: "However, for cases where some loss is acceptable — such as log collection — acks=1 is also reasonable."
Rebuttal: "acks=all increases latency. In systems where throughput matters, this trade-off must be considered."
```

### Applying Steel Man

When addressing opposing views, reconstruct them in their **strongest form** before rebutting.

- ❌ "Some people use acks=0 without really knowing what they're doing"
- ✅ "The strongest case for choosing acks=0 is latency minimization. In systems like real-time sensor data — where overall throughput matters more than the loss of any individual message — this choice is reasonable. However, in the payment system context of this article ..."

### Skeptical stance

- **Avoid overconfidence.** Prefer "X is judged to be ~ under the current architecture" over "X must always be ~"
- **Write falsifiably.** When making a claim, know under what conditions that claim could be wrong.
- **Cite sources.** Let the reader verify.

---

## Post Structure Templates

### Default Structure (PAS-based)

```markdown
# [Title]

## The Problem
[The fundamental problem this post addresses. Make the reader think "I've wondered about this too."]

## Why This Is a Problem
[The concrete consequences of leaving this problem unsolved]

## The Core Principle
[Unpack the key concepts behind the solution step by step]
[Weave in the Claim → Grounds → Qualifier → Rebuttal structure naturally]

## In Practice
[Implementation code, configuration examples, or architecture diagrams]
[Include specific numbers and measured results]

## Trade-offs
[The limits of this approach and alternatives. The key section for intellectual honesty]

## Summary
[One core sentence. What to read next]
```

### Development Journal / Adoption Story Structure (Story Spine-based)

```markdown
# [Title]

## Background
["Once upon a time, we did _____" — the initial situation]

## Discovering the Problem
["But one day, _____ problem appeared" — the inciting incident]

## Exploration
["So we tried _____" — alternatives considered and their pros and cons]

## The Decision and Implementation
["Finally, we solved it with _____" — the final choice rationale + implementation]

## Results
[Before and after comparison. Concrete numbers]

## Retrospective
[What went well, what fell short, what we would do differently]
```

### General Article Structure (Topic-based)

Use this when explaining a broad topic with multiple sub-topics, rather than solving a specific problem or telling a story. Suitable for "X ways to do Y", "Deep dive into Z", or tutorial-style posts.

```markdown
# [Title]

[Opening hook — a concrete scenario or statistic that makes the reader care]
[Brief overview of what this post covers]

---

## 1. [First Topic]

### [Sub-section: Context / Problem]
[Why this topic matters]

### [Sub-section: How It Works / Solution]
[Explanation with code examples, diagrams, or concrete data]

### [Sub-section: Pitfalls / Caveats (optional)]
[Things that can go wrong. What to watch out for]

---

## 2. [Second Topic]

[Same pattern as above — Context → How → Caveats]

---

## [N. Additional Topics as needed]

---

## Summary
[Before/after comparison table or key metrics if applicable]
[Numbered list of core takeaways — one sentence each]

---

> References
```

**When to use which template:**

| Template | Best for | Signal phrases |
|----------|----------|---------------|
| Default (PAS) | Solving a specific problem | "How to fix...", "Why X happens and how to solve it" |
| Development Journal | Sharing a project experience | "How we migrated...", "What I learned building..." |
| General Article | Explaining broad topics with multiple angles | "N ways to...", "Deep dive into...", "Practical guide to..." |

---

## Phase-by-Phase Workflow

### Phase 0: Template Selection (Interactive)

When the user invokes this skill, **always start by presenting the template options** before doing anything else.

Present the following selection prompt to the user:

---

**어떤 스타일로 글을 쓸까요?**

| # | 템플릿 | 설명 | 예시 |
|---|--------|------|------|
| **1** | **Problem-Solution (PAS)** | 특정 문제를 정의하고 해결하는 구조 | "왜 DB 커넥션 풀이 터지는가", "N+1 쿼리 해결법" |
| **2** | **Development Journal** | 프로젝트 경험을 시간순으로 풀어내는 구조 | "Redis 마이그레이션 삽질기", "모노레포 전환 후기" |
| **3** | **General Article** | 넓은 주제를 여러 소주제로 나눠서 설명하는 구조 | "CI/CD 최적화 실전 방법", "Docker 보안 가이드" |

번호로 선택해주세요. (주제를 말씀해주시면 적합한 템플릿을 추천드릴 수도 있습니다)

---

- If `$ARGUMENTS` is "review": skip Phase 0 and go to Phase 3 (Self-Review) on an existing draft file. Ask the user which file to review.
- If the user already specified a topic in `$ARGUMENTS`, suggest the most fitting template but still let them choose.
- If the user picks by number, proceed to Phase 1 with that template.
- If the user describes a topic without picking, recommend a template with reasoning and ask for confirmation.

### Phase 1: Framing the Skeleton

1. **Write the core message in one sentence.** If the reader remembers only one thing from this post, what should it be?
2. **Apply the template chosen in Phase 0.**
3. **Write one line for each section's core point.** This is the skeleton of the post.
4. Show the skeleton to the user and collect feedback.

### Phase 2: Writing the Draft

1. **Save the draft as a local markdown file first.** File path: `blog-drafts/{kebab-case-topic}.md` in the current working directory. Create `blog-drafts/` if it doesn't exist. This works regardless of which project the skill is invoked from.
2. Apply **Style Rules** and **Argumentation Structure** throughout.
3. If content from a `/oh-my-study-with-me:study` session exists, draw on Phase 2–3 conversation content.
4. If code examples are needed, include only the essential parts. Do not paste the entire codebase.
5. If the blog references web sources or makes claims about external tools/technologies, use WebFetch or WebSearch to verify key claims. Include verified references in a "References" section.

### Phase 3: Self-Review + User Confirmation

After completing the draft, review it against this checklist:

**Orwell Check**
- [ ] Are there no tired or cliched metaphors?
- [ ] Is there any place where a longer word was used when a shorter one would do?
- [ ] Have all removable words been removed?
- [ ] Are there any places where passive voice can be made active?
- [ ] Is there any place where a technical term was used when plain language would work?

**Argumentation Check**
- [ ] Does every claim have grounds?
- [ ] Are qualifiers used appropriately? (Is there any overconfidence?)
- [ ] Are opposing views addressed with Steel Man?
- [ ] Are trade-offs explicitly stated?

**Readability Check**
- [ ] Does each paragraph contain only one idea?
- [ ] Is there variation in sentence length?
- [ ] Are only core technical terms written in English? (No overuse?)
- [ ] Are first-use terms explained?

**Kill Your Darlings Check**
- [ ] If there is a sentence you feel proud of, is it for the reader or for yourself?
- [ ] Have all sentences written for self-display been deleted?

Show the review results to the user, then present the revised final version.

**Then ask for user confirmation before proceeding to Phase 4:**

> 초고가 완성되었습니다. (`blog-drafts/{filename}.md`)
> 1. **수정 요청** — 피드백 주시면 반영합니다
> 2. **노션에 발행** — 블로그 DB에 바로 올립니다
> 3. **로컬 파일로 끝내기** — 현재 md 파일로 마무리합니다

Do NOT proceed to Phase 4 until the user explicitly confirms "노션에 발행" or equivalent.

### Phase 4: Publish to Notion

Only execute this phase after user confirmation from Phase 3.

#### Step 1: Confirm target database

Default blog database:
- **Name**: 노션 블로그용 DB
- **ID**: `3133187f-a1c9-80d6-b667-d2d48bb51ec0`

Present the following to the user:

> **어디에 발행할까요?**
> 1. **노션 블로그용 DB** (기본) — 이상훈 기술 블로그
> 2. **다른 DB** — 검색해서 선택합니다

If the user picks 1 or confirms, proceed with the default DB.
If the user picks 2, use `notion-search` to find the target database, show results, and let the user pick.

Once the target DB is confirmed, fetch its schema with `notion-fetch` to get `data_source_id` and property names. Cache these for the session — do not re-fetch.

#### Step 2: Read Notion Markdown spec (once per session)

Fetch the Notion enhanced Markdown spec from `notion://docs/enhanced-markdown-spec` (via ReadMcpResourceTool).
This spec defines what Markdown syntax Notion supports (callouts, toggles, columns, etc.).
Convert the blog content to this format.

**Token-saving**: Only fetch this spec once per session. If already fetched earlier, skip this step.

#### Step 3: Create the page

Use `notion-create-pages` with:
- `data_source_id` from the database schema
- Properties mapped from the blog post:
  - Title property → blog post title
  - Status property → "Draft" (default) or as user prefers
  - Tags/Category property → relevant tags from the content
  - Date property → today's date
  - Other properties → ask the user if unclear
- Content → Notion-compatible Markdown from Step 2

#### Step 4: Confirm

- Show the created page URL to the user.
- Offer: "상태를 Published로 바꾸거나, 태그를 수정하고 싶으면 말씀해주세요."

#### Fallback

If Notion MCP tools are unavailable:
- Inform the user that Notion MCP is not connected.
- The local `blog-drafts/{topic}.md` file is already saved from Phase 2.
- Suggest manual copy-paste to Notion.

---

## Integration with /oh-my-study-with-me:study

When you want to turn learned content into a blog post, invoke this skill separately.
Study notes saved in `study-notes/` from Phase 4 of `/oh-my-study-with-me:study` can be used as source material.
- When given a topic, check `study-notes/` first to see if relevant study notes exist.
- If study notes exist, use the Phase 2–3 conversation content to add depth to the draft.

---

## Important Notes

- Do not copy book content verbatim. Write in the language the user has internalized.
- Do not include unnecessary greetings in the blog post ("Hello, today we're going to learn about ~").
- Do not end with filler phrases like "Thank you" or "I hope this was helpful."
- The tone is "talking with a knowledgeable colleague" — not stiff, but not casual either.
