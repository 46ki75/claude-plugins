---
name: claude-md-improver
description: >
  Audit and improve a project's persistent memory across BOTH CLAUDE.md files
  and the .claude/rules/ directory. Use when the user asks to check, audit,
  review, update, improve, fix, organize, or restructure their CLAUDE.md or
  rules; when CLAUDE.md has grown too long and should be split into path-scoped
  rules; when setting up .claude/rules/; or for "CLAUDE.md maintenance" and
  "project memory optimization". Covers every scope — project ./CLAUDE.md and
  ./.claude/CLAUDE.md, nested CLAUDE.md, ./CLAUDE.local.md, user ~/.claude/
  CLAUDE.md, and rules in .claude/rules/ and ~/.claude/rules/. Trigger even
  without exact wording: "is my CLAUDE.md up to date", "my CLAUDE.md is too
  big", "split this into rules", "review my .claude/rules", "clean up project
  memory" all count.
license: MIT
disable-model-invocation: false
metadata:
  author: "Ikuma Yamashita"
  version: "1.0.0"
---

# CLAUDE.md & Rules Improver

Audit, evaluate, and improve a project's persistent memory so every Claude Code
session starts with accurate, well-scoped context. This skill manages the whole
memory surface, not just the root `CLAUDE.md`:

- **CLAUDE.md files** — `./CLAUDE.md` or `./.claude/CLAUDE.md`, nested
  subdirectory `CLAUDE.md`, and personal `./CLAUDE.local.md`.
- **Rules** — modular, optionally path-scoped instructions in `.claude/rules/`.
- **User scope** — `~/.claude/CLAUDE.md` and `~/.claude/rules/` (only when the
  user asks; never edit these as a side effect of project work).

For the full memory model — load order, scopes, the `paths:` glob mechanism, the
size budget, and what belongs where — read
[references/memory-architecture.md](references/memory-architecture.md). Read it
before scoring or recommending, since the scoring and split logic depend on it.

**This skill can write to memory files.** It always presents a quality report
and gets explicit approval before editing anything.

## Workflow

### Phase 1: Discovery

Find every memory file in scope. Run from the repository root:

```bash
# Project CLAUDE.md family (root, .claude/, nested, and local)
find . -path ./node_modules -prune -o \
  \( -name CLAUDE.md -o -name CLAUDE.local.md \) -print 2>/dev/null | sort

# Project rules
ls -1 .claude/rules/**/*.md .claude/rules/*.md 2>/dev/null
```

Only include user-scope files (`~/.claude/CLAUDE.md`, `~/.claude/rules/`) when
the user explicitly asks about cross-project / personal memory.

Record, for each file: its scope (project / nested / local / user), whether it
is committed to git, line count, and — for rule files — whether it has a
`paths:` frontmatter block.

### Phase 2: Quality assessment

Score each file against the rubric in
[references/quality-criteria.md](references/quality-criteria.md). Beyond the
content-quality criteria, evaluate two memory-structure dimensions that the
original CLAUDE.md-only tooling could not:

- **Scope fit** — is each instruction in the right file? Project-wide facts
  belong in `CLAUDE.md`; topic- or path-specific procedures belong in a rule;
  personal/uncommitted notes belong in `CLAUDE.local.md`.
- **Rule hygiene** — do path-scoped rules have correct, non-overlapping `paths:`
  globs? Are unconditional rules genuinely project-wide, or should they be
  path-scoped to save context? Is anything duplicated between `CLAUDE.md` and a
  rule?

Cross-reference against the actual codebase: do documented commands still exist,
do referenced paths resolve, do `paths:` globs match real directories?

### Phase 3: Quality report (always before editing)

**Output the report before making any change.** Use this shape:

```text
## Project Memory Report

### Summary
- Files found: X (CLAUDE.md: A, rules: B, local: C)
- Average score: X/100
- Total CLAUDE.md size: X lines (budget: <200 per file)
- Files needing update: X

### File-by-file
#### ./CLAUDE.md — Score XX/100 (Grade X), 247 lines
| Criterion        | Score | Notes |
|------------------|-------|-------|
| Commands         | x/20  | ...   |
| Architecture     | x/20  | ...   |
| Non-obvious      | x/15  | ...   |
| Conciseness      | x/15  | ...   |
| Currency         | x/15  | ...   |
| Scope & rules    | x/15  | ...   |
Issues: ...
Recommended: split "Testing" and "Frontend" sections into path-scoped rules.
```

### Phase 4: Recommendations

Propose **targeted** changes only, each shown as a diff with a one-line "why".
Three kinds of recommendation:

1. **Content fixes** — add a missing command, correct a stale path, delete a
   dead instruction, tighten a verbose passage.
2. **Split an oversized CLAUDE.md → path-scoped rules** — when a `CLAUDE.md`
   exceeds ~200 lines or mixes whole-project facts with section-specific
   procedure, extract the section-specific parts into `.claude/rules/<topic>.md`
   with a `paths:` glob. Follow
   [references/splitting-guide.md](references/splitting-guide.md) for detection
   and the exact extraction procedure.
3. **Rule restructuring** — fix or narrow `paths:` globs, merge duplicate rules,
   promote a misplaced personal note to `CLAUDE.local.md`, or pull a duplicated
   block out of `CLAUDE.md` once it lives in a rule.

Use [references/templates.md](references/templates.md) for the canonical shapes
of CLAUDE.md files and rule files.

### Phase 5: Apply with approval

After approval, apply changes with the Edit/Write tools. When a split creates a
new rule file, also remove the migrated content from the source `CLAUDE.md` in
the **same** change so nothing is duplicated across context. Preserve existing
structure and ordering otherwise.

## Common issues to flag

1. **Oversized CLAUDE.md** — over ~200 lines; dilutes adherence. Split into rules.
2. **Misplaced scope** — path-specific procedure living in the global CLAUDE.md.
3. **Duplicated instructions** — same rule in CLAUDE.md and a rule file, or
   across nested CLAUDE.md files.
4. **Bad globs** — `paths:` patterns that match nothing or everything.
5. **Stale content** — commands, paths, or tech versions that no longer exist.
6. **Personal notes in shared files** — sandbox URLs / preferences in committed
   `CLAUDE.md` instead of gitignored `CLAUDE.local.md`.

## Principles

- Concise beats complete: memory is part of every prompt; every line costs
  context and a long file lowers adherence.
- Right file, right scope: put each instruction where it loads when it is needed
  and nowhere else.
- Project-specific over generic: document what is true of _this_ repo, not
  universal best practices.
- Never edit user-scope (`~/.claude/`) memory as a side effect of project work.

Capturing brand-new learnings from the current session is a different job — that
is the companion `revise-claude-md` skill.
