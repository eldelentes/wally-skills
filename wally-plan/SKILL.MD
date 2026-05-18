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

## Phase 2: Recursive A/B/C questions

This is the core of the skill. For every fuzzy area, ask a question with **2-4 concrete options** — not open-ended. Open-ended questions ("how do you want to handle errors?") put the burden on the user. A/B/C questions let them pick fast.

**Pattern:**

> **Question:** [What's the decision to make?]
> **A)** [Concrete option with one-line tradeoff]
> **B)** [Different concrete option with one-line tradeoff]
> **C)** [Third option, if applicable]
> **My recommendation:** [Your pick + one-sentence reasoning]

Always include your recommendation. The user can accept it ("yes, A") or override ("actually B because X"). Either way it's faster than thinking from scratch.

### Use diagrams for architecture

When the decision is about structure — data flow, component boundaries, request lifecycle, state machines — draw it. ASCII diagrams clarify what bullet points obscure, and the user catches "wait, that's not what I meant" much faster from a picture than from prose.

**Example — data flow choice:**

```
A) Direct client → API → DB

    [Client] ──HTTP──> [API] ──SQL──> [DB]


B) Client → API → Queue → Worker → DB (async)

    [Client] ──HTTP──> [API] ──push──> [Queue]
                                          │
                                          ▼
                                      [Worker] ──SQL──> [DB]
```

**Example — state machine for an order:**

```
   ┌─────────┐   submit    ┌────────────┐  approve  ┌──────────┐
   │  draft  │ ──────────> │  pending   │ ────────> │ approved │
   └─────────┘             └────────────┘           └──────────┘
                                  │
                                  │ reject
                                  ▼
                            ┌──────────┐
                            │ rejected │
                            └──────────┘
```

Keep diagrams compact. If they need to scroll horizontally they're too wide — break into multiple smaller diagrams instead.

### Use ASCII mockups for UI decisions

When the decision involves a UI layout — modal vs page, sidebar vs top nav, where a button lives — sketch it in ASCII. It's faster than asking "where should the button go?" and clearer than describing it in words.

**Example — modal vs inline:**

```
A) Modal                              B) Inline expansion

┌─────────────────────────┐           ┌─────────────────────────┐
│ Page                    │           │ Page                    │
│  ┌───────────────────┐  │           │  ┌───────────────────┐  │
│  │  Edit user        │  │           │  │ [▼] John Smith    │  │
│  │  ─────────────    │  │           │  │     ─────────     │  │
│  │  Name: [______]   │  │           │  │     Name: [____]  │  │
│  │  [Save] [Cancel]  │  │           │  │     [Save]        │  │
│  └───────────────────┘  │           │  └───────────────────┘  │
│  (rest dimmed)          │           │  ┌───────────────────┐  │
│                         │           │  │ [▶] Jane Doe      │  │
└─────────────────────────┘           │  └───────────────────┘  │
                                      └─────────────────────────┘
```

The user doesn't need pixel accuracy — they need to see structure to know if it matches what they imagined.

### Ask for screenshots when the user has a reference

If the user mentions "make it look like X" or describes UI in vague terms ("you know, like a settings page"), ask them to paste a screenshot. One sentence:

> "Do you have a screenshot of something similar? Pasting one beats describing it — I can read the layout directly."

Screenshots are especially worth requesting when:
- The user is matching an existing product's UI
- They mention a specific app or website by name
- The UI is non-trivial (dashboards, complex forms, multi-step flows)
- Their description includes hedge words like "kinda", "similar to", "sort of"

If they paste one, study it before proposing the layout. Often the screenshot answers 5 questions you would otherwise have asked.

**Example:**

> User: "I want to add the Fal.ai API to my project."
>
> **How do you want to get results back from Fal.ai?**
> **A)** Polling — simpler, no infra needed, but wastes requests and adds latency.
> **B)** Webhooks — real-time, no wasted requests, but requires a public endpoint (ngrok in dev).
> **C)** Server-sent events — Fal.ai supports this for some endpoints; middle-ground.
>
> **My recommendation:** A for MVP, swap to B once you have a deployed endpoint.

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

Ask: **"Does this look right, or do you want to change anything before I save it?"**

Wait for confirmation. If they want changes, apply them and re-confirm. Don't save a plan they haven't approved.

---

## Phase 4: Save the plan

Save to `.claude/plan/[kebab-case-feature-name].md`. Create the directory if needed (`mkdir -p .claude/plan`). Use the same filename convention as `/wally-brainstorm` so research and plan files line up by name.

After saving, tell the user the path and suggest `/wally-build` as the natural next step if they want execution.

---

## Anti-patterns to avoid

- **Don't ask open-ended questions.** "How should errors work?" makes the user think. "A) retry 3x then fail, B) fail fast and log, C) queue for manual review — I recommend A" lets them pick.
- **Don't skip the assumptions section.** When the plan fails later, it's usually because an unstated assumption was wrong. Stating them surfaces the risk.
- **Don't write tasks that hide decisions.** "Implement caching" is bad. "Add LRU cache to `getUser()` with 5min TTL" is good — every decision visible.
- **Don't save without explicit approval.** The user has to see and accept the plan first.
- **Don't ignore existing research.** Always check `.claude/research/` before asking questions the research already answers.