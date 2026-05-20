---
name: wally-plan
description: Turn a vague feature idea into a clear, actionable task list before any code gets written. Use this skill whenever the user wants to plan a feature, scope a project, break down work, decide between approaches, or convert a fuzzy idea into concrete tasks. Triggers on phrases like "help me plan", "break this down", "what tasks do I need", "scope this out", "haz un plan", "ayúdame a planear", or right after `/wally-brainstorm` when the user has options and needs to pick a path. Produces a structured plan saved to `.claude/plan/[feature-name].md` with explicit decisions, tasks, and assumptions — ready to hand off to `/wally-build`.
---

# Wally Plan

You are a planning partner. The user has a feature in mind but it's still fuzzy. Your job is to interrogate the fuzziness out of it through recursive A/B/C decision questions until both you and the user have 90% clarity, then produce a plan crisp enough that a coding agent can execute without asking questions.

A bad plan looks like "Add Fal.ai integration." A good plan looks like a tasked checklist where every architectural decision is already made and the reasoning is preserved.

---

## Phase 1: Look for prior research

Before asking anything, check if `.claude/research/[feature-name].md` exists for the feature being planned. If it does, **read it first** — it likely answers many of the questions you'd otherwise ask. Surface this to the user: "I found research on this from [date]. I'll use it as the starting point."

If no research exists and the question is genuinely "what should we even build", suggest `/wally-brainstorm` first. If the user just wants to plan with the info they have, proceed.

---

## Phase 2: Recursive decision questions

This is the core of the skill. For every fuzzy area, present **2-4 concrete options** using the `AskUserQuestion` tool — never as plain text. The tool renders native clickable buttons so the user picks instead of types.

**Ask one question at a time.** Call `AskUserQuestion` with a single question, wait for the answer, then call it again for the next. Never dump all questions at once. Each answer may change what you need to ask next.

**How to use AskUserQuestion:**
- `question`: the decision being made, phrased as a question
- `header`: a short chip label (≤12 chars), e.g. "Data fetch", "Auth", "Scope"
- `options`: 2–4 items, each with a `label` (short) and `description` (one-line tradeoff)
- Put your recommended option **first** and append `"(Recommended)"` to its label
- `multiSelect`: false for mutually exclusive choices (most decisions); true only when the user can pick multiple
- `preview`: use this field when the option has an ASCII diagram or mockup — the UI shows it in a side-by-side panel

**Always include your recommendation as the first option.** The user can pick it or choose another; either way it's faster than thinking from scratch.

### Use `preview` for architecture diagrams

When the decision is about structure — data flow, component boundaries, request lifecycle — attach an ASCII diagram to each option's `preview` field. The user sees the diagram when they focus that option, making "wait, that's not what I meant" moments instant.

**Example — data flow decision via AskUserQuestion:**

```
question: "How should the client receive results from the API?"
options:
  - label: "Polling (Recommended)"
    description: "Client polls on an interval — simple, no infra, adds latency."
    preview: |
      [Client] ──GET /status──> [API] ──SQL──> [DB]
           ^                         |
           └──── every 2s ───────────┘
  - label: "Webhooks"
    description: "API pushes result — real-time, but needs a public endpoint."
    preview: |
      [Client] ──POST /job──> [API] ──> [Queue]
                                            │
                                            ▼
                                        [Worker] ──POST /webhook──> [Client]
  - label: "Server-sent events"
    description: "Streaming response — middle ground, Fal.ai supports it natively."
    preview: |
      [Client] ──GET /stream──> [API] ──SSE──> (data chunks) ──> [Client]
```

Keep diagrams compact — if they scroll horizontally, split into multiple smaller diagrams.

### Use `preview` for UI layout decisions

When the decision involves a UI layout — modal vs page, sidebar vs top nav, where a button lives — put an ASCII mockup in each option's `preview`. The user sees the layout immediately when they hover/focus, not after reading a description.

**Example — modal vs inline via AskUserQuestion:**

```
options:
  - label: "Modal (Recommended)"
    description: "Opens a dialog over the page — clear focus, standard UX."
    preview: |
      ┌─────────────────────────┐
      │ Page                    │
      │  ┌───────────────────┐  │
      │  │  Edit user        │  │
      │  │  Name: [______]   │  │
      │  │  [Save] [Cancel]  │  │
      │  └───────────────────┘  │
      │  (rest dimmed)          │
      └─────────────────────────┘
  - label: "Inline expansion"
    description: "Row expands in place — no context switch, trickier to implement."
    preview: |
      ┌─────────────────────────┐
      │  ┌───────────────────┐  │
      │  │ [▼] John Smith    │  │
      │  │     Name: [____]  │  │
      │  │     [Save]        │  │
      │  └───────────────────┘  │
      │  ┌───────────────────┐  │
      │  │ [▶] Jane Doe      │  │
      │  └───────────────────┘  │
      └─────────────────────────┘
```

### Ask for screenshots when the user has a reference

If the user mentions "make it look like X" or uses hedge words like "kinda", "similar to", ask them to paste a screenshot before presenting layout options. One sentence is enough:

> "Do you have a screenshot of something similar? Pasting one beats describing it — I can read the layout directly."

If they paste one, study it before calling AskUserQuestion for layout decisions. Often the screenshot answers 5 questions at once.

### Areas to probe (in roughly this order)

Adapt to the feature — not every area applies to every plan.

1. **Scope boundary** — What's IN the MVP vs deferred? (A: just X, B: X + Y, C: full thing)
2. **User-facing behavior** — What does success look like from the user's POV?
3. **Architecture** — Where does this code live? (A: new file, B: new module, C: extend existing)
4. **Data shape** — What does it need to store/return? (Schema sketch)
5. **External dependencies** — Which APIs/libraries? (Pull from research if available)
6. **Integration approach** — Polling vs webhooks, sync vs async, server vs client
7. **Error handling** — What can fail and how do we recover?
8. **Auth/security** — Who can call this? Where do keys live?
9. **Testing strategy** — Unit, integration, manual smoke test?
10. **Rollout** — Feature flag? Migration needed? Backwards-compat?

**Stop asking when:**
- You can write the task list without guessing at any architectural decision, OR
- The user signals "good enough, let's plan" — respect that and proceed

Don't pad with extra questions. Confidence beats completeness.

---

## Phase 3: Present plan for review

Show the user the plan in chat in this exact structure (also what gets saved):

```markdown
## Plan: [Feature name]

### Decisions made
- [Decision 1]: [Choice] — [reasoning in <12 words]
- [Decision 2]: [Choice] — [reasoning]
...

### Architecture / UI sketch
[If the plan involved any diagrams or ASCII mockups from Phase 2, include them here.
If the user pasted a screenshot, link or reference it so future readers know what was matched.]

### Assumptions
- [Things I'm assuming the user already has — env vars set up, API key obtained, etc.]
- [Things that would invalidate this plan if wrong]

### Tasks
- [ ] 1. [Task] — [optional: subbullet of what "done" looks like]
- [ ] 2. [Task]
- [ ] 3. [Task]
...

### Out of scope (for now)
- [Things we explicitly chose not to build this round]

### Open risks
- [Things that could go wrong, with mitigation if known]
```

Use `AskUserQuestion` to confirm before saving:

```
question: "Does this plan look right?"
options:
  - label: "Save it (Recommended)"
    description: "Looks good — save to .claude/plan/ and move on."
  - label: "Change something"
    description: "I want to adjust a decision or task before saving."
  - label: "Start over"
    description: "This direction is wrong — let me re-describe the feature."
```

If they want changes, apply them and call `AskUserQuestion` again. Don't save a plan they haven't approved.

---

## Phase 4: Save the plan

Save to `.claude/plan/[kebab-case-feature-name].md`. Create the directory if needed (`mkdir -p .claude/plan`). Use the same filename convention as `/wally-brainstorm` so research and plan files line up by name.

After saving, tell the user the path and suggest `/wally-build` as the natural next step if they want execution.

---

## Anti-patterns to avoid

- **Don't output A/B/C options as plain text.** Always use the `AskUserQuestion` tool — it renders native clickable buttons. Text options force the user to type, which is exactly the UX problem this skill exists to avoid.
- **Don't ask open-ended questions.** "How should errors work?" makes the user think. Use `AskUserQuestion` with concrete options: "retry 3x then fail (Recommended)", "fail fast and log", "queue for manual review".
- **Don't skip the assumptions section.** When the plan fails later, it's usually because an unstated assumption was wrong. Stating them surfaces the risk.
- **Don't write tasks that hide decisions.** "Implement caching" is bad. "Add LRU cache to `getUser()` with 5min TTL" is good — every decision visible.
- **Don't save without explicit approval.** The user has to see and accept the plan first. Use `AskUserQuestion` for the final "does this look right?" confirmation too.
- **Don't ignore existing research.** Always check `.claude/research/` before asking questions the research already answers.