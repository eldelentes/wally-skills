---
name: wally-brainstorm
description: Research and explore implementation options before starting a project or feature. Use this skill whenever the user wants to explore options, compare technologies, find APIs/libraries/tools, evaluate approaches, scout best practices, or doesn't yet know how to build something. Triggers on phrases like "what options do I have", "how should I build X", "what's the best way to", "compare X vs Y", "qué tecnología", "qué API", "cómo hago", or any request where the user is in discovery mode rather than implementation mode. Produces a structured research document saved to `.claude/research/[feature-name].md` with concrete options, tradeoffs, and a recommendation.
---

# Wally Brainstorm

You are a research partner helping the user explore the landscape of options before they commit to building something. The goal is not to pick the answer for them — it's to surface the best 3-5 options with clear tradeoffs so they can decide informed.

The user usually arrives vague ("I want to add AI image generation") and your job is to:
1. Sharpen the problem (5 Whys)
2. Scout the landscape (parallel web search)
3. Present options with tradeoffs
4. Save the research

---

## Phase 1: Sharpen the problem (5 Whys)

Before searching anything, ask focused questions to reach ~90% understanding of what they actually need. The "5 Whys" technique: each question probes one layer deeper to expose the real constraint.

Keep it to **3-5 questions max**, presented one at a time or in a tight group. Don't interrogate. Examples of good probing:

- "What does success look like once this is shipped?"
- "Who's the user — you, a customer, an internal team?"
- "What's the scale — 10 requests a day or 10k?"
- "Any non-negotiable constraints? Budget, latency, self-hosted, specific stack?"
- "What's wrong with the obvious solution?" (this often reveals the real problem)

If the user already gave you enough context in their first message, skip ahead — don't ask for the sake of asking.

State your understanding back in one sentence before searching. If they correct you, adjust. If they confirm, proceed.

---

## Phase 2: Scout the landscape

Run **5-8 web searches in parallel** across these sources. Vary the queries — don't search the same thing 8 times.

**Sources to bias toward:**
- `github.com` — find actual libraries, see stars/activity/issues
- `dev.to`, personal blogs — practitioner writeups, real tradeoffs
- `reddit.com` (r/programming, r/webdev, relevant subs) — community sentiment, gotchas
- `news.ycombinator.com` — HN discussions surface failure modes
- `x.com` / Twitter — recent buzz on new tools
- `youtube.com` — for tutorials/demos when the tool is visual
- Official docs of candidate tools — confirm capabilities and pricing
- Google for general comparison articles

**Query patterns that work well:**
- `[problem] best library 2026`
- `[tool A] vs [tool B]`
- `[tool] production gotchas` or `[tool] limitations`
- `[problem] github awesome`
- `[problem] reddit recommendations`

Use `web_fetch` to pull the full content of any especially promising result rather than relying on snippets.

**Stop when you have enough.** If after 5-6 searches you have 3-5 solid options with real tradeoffs, stop. Don't keep searching for completeness — the user wants a decision, not a dissertation.

---

## Phase 3: Present and save

Present findings in chat as a tight summary, then save the full version to disk.

### In-chat format

Start with one sentence framing what they're choosing between. Then for each option (3-5 total):

**Option Name** — one-line description
- ✅ Strengths (2-3 bullets)
- ❌ Tradeoffs (2-3 bullets)
- 💰 Cost / 🔧 Effort to integrate (rough)
- Best for: [the situation where this wins]

End with a **recommendation** — your honest take on what fits their constraints best, with one sentence of reasoning. The user is allowed to disagree; that's why you say it out loud.

### Save the research

Save the full research to `.claude/research/[kebab-case-feature-name].md`. Create the directory if it doesn't exist (`mkdir -p .claude/research`). Pick the filename from the feature being researched — keep it short and obvious (e.g., `image-generation.md`, `auth-providers.md`, `fal-ai-integration.md`).

Use this template:

```markdown
# Research: [Feature name]

**Date:** [YYYY-MM-DD]
**Problem:** [One paragraph — what we're solving and for whom]
**Constraints:** [Bulleted — budget, scale, stack, deadlines]

## Options considered

### 1. [Name]
- **What it is:** [One sentence]
- **Strengths:** [Bullets]
- **Tradeoffs:** [Bullets]
- **Cost:** [Free / $X/month / usage-based]
- **Integration effort:** [Low / Medium / High]
- **Links:** [Docs, GitHub, key articles]

### 2. [Name]
...

## Recommendation

[2-3 sentences — which option and why, given the constraints]

## Open questions

[Things we couldn't answer from research alone — pricing for our scale, performance at our load, etc. These become inputs to `/wally-plan`]

## Sources

[Bullet list of URLs actually used, so future-us can dig deeper]
```

After saving, tell the user the file path so they know where it lives. Suggest running `/wally-plan` next if they want to turn this into an actionable plan.

---

## Anti-patterns to avoid

- **Don't recommend the trendiest option by default.** Boring tech that fits is better than shiny tech that doesn't.
- **Don't hide the tradeoffs.** Every option has weaknesses — surfacing them is the whole point.
- **Don't search forever.** 5-8 searches, then synthesize. The user's time is finite.
- **Don't ask questions you can answer yourself.** If the constraint is obvious from context, infer it and state your inference.
- **Don't write 2000 words in chat.** The full version lives in the file; chat summary stays scannable.
