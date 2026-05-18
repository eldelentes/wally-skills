# wally-skills

A collection of Claude Code skills that cover the full lifecycle of a software project — from ideation and research to implementation and learning from experience.

Each skill is a focused, opinionated agent that handles one phase of the development workflow. They're designed to chain together naturally: brainstorm → setup → build → learn.

---

## Skills

### `/wally-brainstorm`

Research and explore implementation options before committing to building something.

Arrives at a structured research document saved to `.claude/research/[feature-name].md` with concrete options, tradeoffs, and a recommendation.

**When to use:**
- You know what you want to build but not how
- You want to compare technologies, libraries, or APIs
- You're in discovery mode and need to see the landscape

**Trigger phrases:** "what options do I have", "how should I build X", "compare X vs Y", "qué tecnología", "qué API"

**What it does:**
1. Sharpens the problem with focused questions (5 Whys)
2. Runs 5–8 parallel web searches across GitHub, Reddit, HN, official docs, and dev blogs
3. Presents 3–5 options with strengths, tradeoffs, cost, and effort
4. Gives an honest recommendation based on your constraints
5. Saves the full research to `.claude/research/[feature].md`

**Example:**
```
/wally-brainstorm I want to add real-time notifications to my Next.js app
```

Output: a comparison of WebSockets, SSE, Pusher, Ably, and Supabase Realtime — with a recommendation based on your scale and stack.

---

### `/wally-plan`

Turn a vague feature idea into a clear, actionable task list before any code gets written.

Arrives at a structured plan saved to `.claude/plan/[feature-name].md` with explicit decisions, tasks, and assumptions — ready to hand off to `/wally-build`.

**When to use:**
- You've done the research (or already know what you want) and need to scope it into tasks
- You want to lock in architectural decisions before touching code
- Right after `/wally-brainstorm` when you have options and need to pick a path

**Trigger phrases:** "help me plan", "break this down", "what tasks do I need", "scope this out", "haz un plan", "ayúdame a planear"

**What it does:**
1. Checks `.claude/research/[feature-name].md` for prior research and uses it as a starting point
2. Asks recursive A/B/C questions for every fuzzy area — always with a recommendation, never open-ended
3. Uses ASCII diagrams for architecture decisions and ASCII mockups for UI layout decisions
4. Presents the full plan for your approval before saving anything
5. Saves the approved plan to `.claude/plan/[feature-name].md`

**Example:**
```
/wally-plan add real-time notifications
```

Output: decisions locked (polling vs webhooks, where state lives, auth scope), assumptions listed, and a numbered task list the builder can execute without asking questions.

---

### `/wally-setup`

Bootstrap a new project from scratch: create the GitHub repo, scaffold the chosen stack, find relevant skills, and write a custom README.

**When to use:**
- Starting a new project from zero
- You want a real repo with proper scaffolding, not a local folder

**Trigger phrases:** "start a new project", "create a new repo", "scaffold X", "set up a new app", "crear un proyecto nuevo"

**What it does:**
1. Gathers project name, stack, description, and visibility
2. Runs preflight checks (`gh auth status`, `git`)
3. Creates a private GitHub repo via `gh` CLI and clones it
4. Scaffolds the stack using the canonical bootstrapper (e.g. `create-next-app`, `uv init`)
5. Discovers and installs relevant skills from the skills.sh ecosystem
6. Writes a custom README tailored to the actual scaffolded structure
7. Commits and pushes everything

> This skill has real side effects. It confirms with you before creating the repo or pushing.

**Example:**
```
/wally-setup project: task-tracker, stack: Next.js + Tailwind + Supabase, private
```

---

### `/wally-build`

Execute a feature implementation end-to-end — branch, code, test, commit, push, PR.

**When to use:**
- You have a plan (from `/wally-brainstorm` or your own head) and want to ship it
- You want the full loop: branch → code → verify → commit → PR

**Trigger phrases:** "build this", "implement", "ship it", "create the PR", "haz esta feature", "implementa"

**What it does:**
1. Reads the plan from `.claude/plan/[feature-name].md` (or works from a verbal description)
2. Creates a `feat/[name]` branch off `main`
3. Defines a testing strategy before writing any implementation
4. Decomposes tasks into parallel (independent) and sequential (dependent) work
5. Spawns parallel subagents for independent tasks to save time
6. Runs lint, typecheck, and tests before committing
7. Commits with conventional commit messages (`feat:`, `fix:`, `chore:`, etc.)
8. Pushes and opens a PR via `gh pr create` with a structured description

> Confirms with you before pushing or opening the PR. Never auto-merges.

**Example:**
```
/wally-build implement the auth flow from the plan
```

---

### `/wally-learn`

Extract durable patterns from the current conversation and propose updates to `CLAUDE.md`.

**When to use:**
- At the end of a productive session where you corrected Claude or established conventions
- When you want Claude to remember something for all future conversations
- When you notice Claude making the same mistake repeatedly

**Trigger phrases:** "save what we learned", "update CLAUDE.md", "remember this for next time", "aprende de esto", "guarda esto"

**What it does:**
1. Scans the conversation for patterns worth capturing: repeated corrections, stated preferences, architectural decisions, gotchas
2. Categorizes them: tech stack, code style, architecture, forbidden patterns, workflow, domain knowledge
3. Presents proposed additions for your approval — you can accept, edit, or reject each one
4. Writes only what you explicitly approve to `CLAUDE.md`
5. Shows you the diff so you can verify it landed correctly

> Never writes to `CLAUDE.md` without explicit confirmation. Silent writes are an anti-pattern.

**Example:**
```
/wally-learn
```

After a session where you corrected Claude to use Vitest instead of Jest, avoid Redux, and always use kebab-case for file names — it surfaces those three as candidates and asks which to keep.

---

## How they chain together

```
/wally-brainstorm  →  research saved to .claude/research/
/wally-plan        →  decisions locked, tasks saved to .claude/plan/
/wally-setup       →  repo created, stack scaffolded
/wally-build       →  feature branch, code, PR
/wally-learn       →  patterns saved to CLAUDE.md
```

You don't have to use all of them — each skill works standalone. But they're designed so the output of one becomes the input of the next.

---

## Installation

```bash
npx skills add https://github.com/eldelentes/wally-skills --skill wally-brainstorm
npx skills add https://github.com/eldelentes/wally-skills --skill wally-plan
npx skills add https://github.com/eldelentes/wally-skills --skill wally-setup
npx skills add https://github.com/eldelentes/wally-skills --skill wally-build
npx skills add https://github.com/eldelentes/wally-skills --skill wally-learn
```

Or install all at once:

```bash
npx skills add https://github.com/eldelentes/wally-skills
```
