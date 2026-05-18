---
name: wally-learn
description: Extract durable patterns, preferences, and corrections from the current conversation and propose updates to CLAUDE.md. Use this skill whenever the user wants to capture what was learned, update their Claude memory/rules, save preferences, or "make Claude remember this". Triggers on phrases like "save what we learned", "update CLAUDE.md", "remember this for next time", "aprende de esto", "guarda esto". Reviews the current conversation for repeated corrections, stated preferences, architectural decisions, and gotchas — then presents the extracted patterns for user approval before writing anything to CLAUDE.md. Never writes without explicit confirmation.
---

# Wally Learn

You read back over the current conversation and extract patterns worth remembering — corrections the user made repeatedly, preferences they stated, decisions they explained the reasoning behind, gotchas about their codebase, anything that would have made the conversation go better if it had been known upfront.

You then **propose** those patterns to the user for approval, and only write to `CLAUDE.md` after they confirm. The supervised flow is the whole point: Claude is bad at knowing what's worth remembering, and silent writes to CLAUDE.md compound that.

---

## Phase 1: Scan the conversation

Read back through the conversation. Look for these signals:

**Strong signals (almost always worth capturing):**
- The user said the same correction more than once ("again, I want all windows maximized")
- The user explained the why behind a preference ("use camelCase because our backend does")
- The user pointed out something Claude got wrong about the codebase ("we don't use Redux here, we use Zustand")
- The user established a constraint ("never use `any` in TypeScript")

**Medium signals (capture if they're general, skip if one-off):**
- A style preference stated once
- A tool or library choice ("we use Vitest, not Jest")
- A naming convention

**Weak signals (probably skip):**
- A one-off opinion about this specific feature
- Something that's already in CLAUDE.md
- Anything tied to a single ephemeral task

When in doubt, lean toward proposing it and let the user reject it. Cheap to skip, expensive to miss.

---

## Phase 2: Categorize and dedupe

Group the candidates into categories so they slot cleanly into CLAUDE.md:

- **Tech stack** (we use X, not Y)
- **Code style** (formatting, naming, structure preferences)
- **Architectural patterns** (how we organize files, what abstractions we prefer)
- **Forbidden patterns** (things to never do)
- **Workflow** (how we ship, review, deploy)
- **Domain knowledge** (project-specific concepts Claude needs to know)

If CLAUDE.md already exists, read it first and skip anything already covered. Don't propose duplicates.

---

## Phase 3: Present for review

Show the user the proposed additions in this format:

```markdown
## Proposed additions to CLAUDE.md

### [Category]
- **[Pattern in one line]**
  - *Why:* [Where this came from in the conversation]
  - *Source:* [Brief context — what triggered this]

### [Another category]
- **[Pattern]**
  - *Why:* [...]
  - *Source:* [...]
```

For each item, the user can:
- ✅ **Accept** as-is
- ✏️ **Edit** the wording
- ❌ **Reject** (not worth capturing, or too specific)

Ask: **"Which of these should I add? You can say 'all', 'none', or list specific ones to keep/edit."**

Wait for explicit response. Don't proceed without confirmation.

---

## Phase 4: Write to CLAUDE.md

Once the user confirms, write to `CLAUDE.md` at the project root (or `~/.claude/CLAUDE.md` if the user specifies global).

**Rules for writing:**

1. **Append, don't overwrite.** If CLAUDE.md exists, add new items to the appropriate section. If a section doesn't exist, create it.
2. **Keep entries terse.** One-line bullets. CLAUDE.md is read on every session — every word costs context.
3. **Use imperative voice.** "Use Vitest" not "We tend to prefer Vitest sometimes".
4. **Group by category.** Don't append a flat list — keep the structure scannable.

Suggested CLAUDE.md structure if creating fresh:

```markdown
# Project conventions

## Stack
- [items]

## Code style
- [items]

## Architecture
- [items]

## Never do
- [items]

## Workflow
- [items]

## Domain knowledge
- [items]
```

After writing, show the user the diff (or the section that changed) so they can verify it landed correctly.

---

## Phase 5: Suggest a quick sanity check

Tell the user: "Next conversation, these should kick in automatically. If you notice me still getting any of these wrong, run `/wally-learn` again and we'll sharpen the wording."

---

## Anti-patterns to avoid

- **Don't write to CLAUDE.md without explicit approval.** Even the "obviously good" patterns need a yes.
- **Don't capture ephemeral context.** "Use the blue button for this screen" is not a durable pattern — that's a feature decision, not a convention.
- **Don't bury the lede in long entries.** "Use Vitest" beats "When writing tests for this project, please use the Vitest testing framework instead of Jest because the team has standardized on it."
- **Don't duplicate what's already there.** Always read CLAUDE.md first.
- **Don't propose 20 items.** If you find that many, you're including weak signals — filter harder. 3-7 is a healthy range per session.
- **Don't editorialize on why the user is right.** Just capture the rule. The reasoning lives in the conversation; CLAUDE.md is for the rule itself.
