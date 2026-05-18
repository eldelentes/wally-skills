---
name: wally-setup
description: Bootstrap a brand-new project with repo creation, stack-appropriate scaffolding, and a custom README. Use this skill whenever the user wants to start a new project, initialize a repo, scaffold a new codebase, set up a fresh app, or kick off something from scratch. Triggers on phrases like "start a new project", "create a new repo", "scaffold X", "set up a new app", "crear un proyecto nuevo", "iniciar repo". Creates a private GitHub repo via `gh` CLI, scaffolds the chosen stack, finds and installs relevant skills from skills.sh, and writes a custom README. Has real side effects (creates a repo, pushes code) so requires user confirmation before each destructive step.
disable-model-invocation: true
---

# Wally Setup

You bootstrap a new project. This skill has **real side effects** — it creates a GitHub repository, scaffolds files, and pushes code. Because of that, confirm with the user before each destructive step. Never assume.

The goal is to compress the boring "first 30 minutes of a new project" into a few minutes, while leaving the user fully in control of decisions.

---

## Phase 1: Gather inputs

Ask the user (if not already provided) for:

1. **Project name** — kebab-case, will be the repo name and the local folder
2. **Tech stack** — language + framework (e.g., "Next.js + Tailwind + Supabase", "FastAPI + Postgres", "Swift + SwiftUI")
3. **One-line description** — for the README and `gh` repo description
4. **Visibility** — confirm private (default) or public

Bundle these into one message with concrete suggestions if you can infer from context. Don't ask for what you already know.

---

## Phase 2: Verify environment

Before touching anything, run preflight checks:

```bash
gh auth status     # confirm gh CLI is authenticated
git --version      # confirm git installed
```

If `gh` isn't authenticated, stop and tell the user to run `gh auth login` first. Don't try to authenticate on their behalf.

---

## Phase 3: Create the repo

Confirm with the user **before** running this. Show them the exact command you're about to execute.

```bash
gh repo create [name] --private --description "[description]" --clone
cd [name]
```

After cloning, you're inside the project directory. From here on, all commands run from there.

---

## Phase 4: Scaffold the stack

Scaffold according to the chosen stack. Use the canonical bootstrapper for each — don't hand-roll structure that an official tool already does well.

**Common scaffolders:**
- Next.js: `npx create-next-app@latest . --typescript --tailwind --app`
- Vite + React: `npm create vite@latest . -- --template react-ts`
- Astro: `npm create astro@latest .`
- SvelteKit: `npm create svelte@latest .`
- FastAPI: create `pyproject.toml` with `uv init` or `poetry init`, scaffold `app/main.py`
- Expo: `npx create-expo-app@latest .`
- SwiftUI: scaffold manually (no official CLI for app projects)

If the stack is unfamiliar or has multiple bootstrappers, ask the user which one to use rather than guessing.

After scaffolding:
```bash
git add -A
git commit -m "chore: initial scaffold"
git push -u origin main
```

Confirm with the user before the push.

---

## Phase 5: Find and install relevant skills

Use the **`find-skills` skill from Vercel Labs** as the discovery layer — it wraps the Skills CLI and knows how to search the skills.sh ecosystem properly.

### One-time bootstrap (if not already installed)

Check whether `find-skills` is available in this environment. If not, install it once:

```bash
npx skills add https://github.com/vercel-labs/skills --skill find-skills
```

This only needs to happen once per machine; subsequent projects reuse it.

### Discovering skills for this project

With `find-skills` available, hand off the discovery step. Tell the model running `find-skills`:

> "Find skills relevant to a project using [stack — e.g., Next.js + Tailwind + Supabase]. Surface the top 3-5 candidates by install count and source reputation, with install commands."

Or run the underlying CLI directly:

```bash
npx skills find [stack keyword]      # e.g. npx skills find nextjs
npx skills find [framework] tailwind
npx skills find [language] testing
```

### Present to the user and install

Show the user the top 3-5 candidates with one-line descriptions and let them pick. For each they accept, install with:

```bash
npx skills add [package] --skill [skill-name]
```

The user can also skip this phase entirely if they want a clean start. Don't install anything they didn't explicitly approve.

---

## Phase 6: Write a custom README

Generate a `README.md` tailored to the project — not boilerplate. Include:

```markdown
# [Project name]

[One-paragraph description: what it does, who it's for, what's special about it]

## Stack
- [Each major piece, one line]

## Getting started

\`\`\`bash
[install command]
[run dev command]
\`\`\`

## Project structure

[Brief overview of key directories]

## Conventions

[Anything from `.claude/skills/` worth highlighting — coding style, commit format, etc.]
```

Avoid the generic "this project was bootstrapped with X" filler. Read the actual scaffolded structure and write about what's there.

Commit the README:
```bash
git add README.md
git commit -m "docs: add custom README"
git push
```

---

## Phase 7: Wrap up

Tell the user:
- Repo URL (`gh repo view --web` to confirm)
- Local path
- Suggested next step: `/wally-plan` for the first feature, or `/wally-build` if they already have a plan

---

## Anti-patterns to avoid

- **Don't run destructive commands without confirmation.** Repo creation, force pushes, anything irreversible needs an explicit OK.
- **Don't guess the stack.** If the user says "a React app", ask: Vite or Next? TypeScript or JS? Tailwind or not?
- **Don't write generic README boilerplate.** A README that doesn't tell you what the project does is worse than no README.
- **Don't install skills the user didn't ask for.** Surface candidates, let them choose.
- **Don't proceed if `gh` isn't authenticated.** Stop and tell the user; don't try workarounds.
