---
name: wally-build
description: Execute a feature implementation end-to-end — branch, code, test, commit, push, PR. Use this skill whenever the user wants to implement a feature, build something based on a plan, ship code, or convert a plan into a pull request. Triggers on phrases like "build this", "implement", "ship it", "create the PR", "haz esta feature", "implementa", or right after `/wally-plan` when the user has an approved plan. Reads from `.claude/plan/[feature-name].md` if available, spawns parallel subagents for independent tasks, commits with conventional commit messages, pushes, and opens a PR via `gh`. Has real side effects (writes code, pushes branches, opens PRs) so requires user confirmation at key points.
disable-model-invocation: true
---

# Wally Build

You execute a feature implementation. This skill takes a plan (ideally from `/wally-plan`) and turns it into committed code with an open PR. Real side effects — confirm with the user before pushing or opening the PR.

The skill follows TDD-ish principles: define tests, then implement, then verify. When tasks are independent, parallelize them with subagents. When they depend on each other, do them in sequence.

---

## Phase 1: Load the plan

Look for `.claude/plan/[feature-name].md` for the feature being built. Read it fully. The plan tells you:
- What decisions are already made (don't re-litigate)
- What tasks exist (your work breakdown)
- What's explicitly out of scope (don't drift into it)
- What assumptions were made (verify they still hold)

If no plan exists and the user wants to build directly from a verbal description, that's fine — but suggest running `/wally-plan` first if the feature has any complexity. Quick fixes don't need a plan; multi-file features usually do.

---

## Phase 2: Create the feature branch

Pick a branch name from the feature: `feat/[kebab-case-name]` for new features, `fix/[name]` for bugs, `chore/[name]` for non-functional work.

```bash
git checkout main && git pull
git checkout -b feat/[name]
```

If the working tree is dirty, stop and ask the user how to handle it — don't stash or discard their work silently.

---

## Phase 3: Define the testing strategy

Before writing any implementation code, decide what "done" looks like in testable terms. Pull this from the plan if it's there; otherwise sketch it now:

- **Unit tests** for pure functions / business logic
- **Integration tests** for things that cross boundaries (DB, API, file system)
- **Manual smoke test** steps for UI changes

State the testing strategy out loud — even if the user said "skip tests for now", at least name what you're skipping so it's an informed call.

---

## Phase 4: Decompose and execute

Break the plan's task list into:

- **Independent tasks** — can run in parallel (e.g., "create the API route" and "create the UI component" if neither imports the other yet)
- **Sequential tasks** — depend on something previous (e.g., "wire the UI to the API" requires both to exist)

### Parallel execution with subagents

For independent tasks, spawn subagents using the Task tool. Each subagent gets:
- The specific task description (one task, not the whole plan)
- The relevant file paths it should read/write
- Constraints: which files NOT to touch, which patterns to follow
- What "done" looks like for this slice

Wait for all parallel subagents to finish before moving to the sequential integration step.

### Sequential execution

For dependent tasks, do them yourself in order. After each task:
- Run tests for that piece if applicable
- Verify the code compiles / lints / typechecks
- Move to the next task only if the previous one is green

---

## Phase 5: Verify and self-review

Before committing, run the project's checks:

```bash
# Whatever applies to the project:
npm test
npm run lint
npm run typecheck
# or pytest, cargo test, etc.
```

If anything fails, fix it before proceeding. Don't paper over with `--no-verify` flags or skipped tests.

Then do a quick self-review by reading `git diff` and asking:
- Did I do exactly what the plan said, no more no less?
- Any debug prints, commented-out code, TODO markers I left in?
- Are the changes coherent — would someone reviewing this understand the story?

---

## Phase 6: Commit

Use conventional commits format. One commit per logical unit — not one giant commit, not 50 micro-commits.

```bash
git add [files]
git commit -m "feat(scope): short summary

[optional body explaining why, not what]"
```

Format reference:
- `feat:` new feature
- `fix:` bug fix
- `chore:` non-functional (deps, config)
- `refactor:` no behavior change
- `docs:` documentation only
- `test:` tests only

---

## Phase 7: Push and open PR

Confirm with the user before pushing. Then:

```bash
git push -u origin feat/[name]
gh pr create --title "[type]: [short title]" --body "$(cat <<'EOF'
## Summary
[2-3 sentences — what changed and why]

## Plan reference
Based on `.claude/plan/[feature-name].md`

## Changes
- [Bullet per logical change]

## Testing
- [How to verify this works]

## Out of scope
[What's intentionally not in this PR — links to follow-up if planned]
EOF
)"
```

After the PR is open, share the URL with the user and stop. The PR is theirs to review and merge. Don't auto-merge.

---

## Anti-patterns to avoid

- **Don't drift from the plan.** If you find yourself wanting to "also fix this one other thing", note it for a follow-up PR. Scope creep is what makes PRs unreviewable.
- **Don't skip verification because tests are slow.** A slow test is faster than a regression.
- **Don't push without confirmation.** Real branches, real PRs, real reviewers' time.
- **Don't write one giant commit at the end.** Logical units mean someone can `git blame` later and understand why a line exists.
- **Don't parallelize tasks that depend on each other.** Subagents in parallel are powerful for genuinely independent work — they cause chaos when shared state is involved.
- **Don't `--no-verify` past failing hooks.** If a hook is wrong, fix the hook; if the code is wrong, fix the code.
