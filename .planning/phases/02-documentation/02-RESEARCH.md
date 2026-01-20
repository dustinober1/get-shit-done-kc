# Phase 02: Documentation - Research

**Researched:** 2026-01-19
**Domain:** Technical documentation for CLI tool
**Confidence:** HIGH

## Summary

This phase documents the Quick Mode feature (`/gsd:quick`) across three files: help.md (command reference), README.md (user-facing overview), and GSD-STYLE.md (style guide for GSD patterns).

The documentation phase differs from code implementation phases. The "stack" is the existing documentation infrastructure itself. Research findings here focus on understanding existing documentation patterns, tone conventions, and placement strategies.

**Primary recommendation:** Use existing documentation patterns as templates — help.md follows the reference format established by other commands, README.md adds Quick Mode section after "How It Works", GSD-STYLE.md adds descriptive Quick Mode Patterns section following the existing format.

## Standard Stack

This is a documentation phase — no external libraries or tools. The "stack" consists of:

| Component | Purpose | Why Standard |
|-----------|---------|--------------|
| help.md | Command reference (displayed by `/gsd:help`) | Central reference for all GSD commands |
| README.md | User-facing project documentation | Primary onboarding document |
| GSD-STYLE.md | Style guide for GSD meta-patterns | Reference for contributors and power users |
| CHANGELOG.md | Version history | Already documents Quick Mode in v1.7.0 |

**Installation:** N/A (no packages to install)

## Architecture Patterns

### help.md Section Placement

**Current Quick Mode section location:** After "Execution" section, before "Roadmap Management"

This placement follows the logical flow:
1. Quick Start
2. Staying Updated
3. Core Workflow
4. Project Initialization
5. Phase Planning
6. **Quick Mode** (here - immediate alternative to full planning)
7. Roadmap Management

**Pattern:** The section structure follows the user journey from getting started to advanced usage.

### help.md Content Pattern

From `/gsd:quick` entry in help.md (lines 132-146):

```markdown
### Quick Mode

**`/gsd:quick`**
Execute small, ad-hoc tasks with GSD guarantees but skip optional agents.

Quick mode uses the same system with a shorter path:
- Spawns planner + executor (skips researcher, checker, verifier)
- Quick tasks live in `.planning/quick/` separate from planned phases
- Updates STATE.md tracking (not ROADMAP.md)

Use when you know exactly what to do and the task is small enough to not need research or verification.

Usage: `/gsd:quick`
Result: Creates `.planning/quick/NNN-slug/PLAN.md`, `.planning/quick/NNN-slug/SUMMARY.md`
```

**Pattern elements:**
- Name with backticks
- One-line description
- Bullet points explaining the key differences
- "Use when" guidance
- Usage syntax
- Result expectation

### README.md Section Pattern

Quick Mode appears in README.md in TWO locations per STATE.md decision:

1. **Dedicated "Quick Mode" section** (after "How It Works" section, ~line 300-323):
   - Full `## Quick Mode` heading
   - Code block showing command invocation
   - Descriptive tagline: "For ad-hoc tasks that don't need full planning"
   - Bullet points explaining what quick mode gives you
   - Use case examples
   - Output format

2. **Utilities table entry** (in command reference tables section, ~line 457):
   - Table format matching other utility commands
   - One-line description

### GSD-STYLE.md Pattern

GSD-STYLE.md sections follow a consistent structure (based on existing sections like "TDD Plans", "Progressive Disclosure"):

```markdown
## [Section Name]

[One paragraph explaining what this is]

### Subsection

[Content with code examples if applicable]

[Code example if needed]
```

For Quick Mode Patterns, follow the existing section added in v1.7.0 (lines 379-444):
- Descriptive tone (not prescriptive rules)
- Explains what quick mode does
- Lets reader decide when to use it
- Code examples showing structure

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Documentation formatting | Custom markdown patterns | Existing section structures | Consistency across docs helps users scan quickly |
| Quick mode explanation | Writing from scratch | Content from `/gsd:quick` command file | The command file already has accurate description |
| Placement decisions | Guessing where content goes | Following logical flow of user journey | Existing docs are organized by workflow order |

**Key insight:** Documentation is about discoverability. Users should find Quick Mode documentation where they naturally look: when they want an alternative to full phase planning (help.md), when learning the system (README.md), or understanding GSD patterns (GSD-STYLE.md).

## Common Pitfalls

### Pitfall 1: Over-documenting in help.md

**What goes wrong:** help.md becomes verbose, defeating its purpose as a quick reference.

**Why it happens:** Desire to be thorough leads to explaining too much.

**How to avoid:** help.md is a reference, not a tutorial. Keep entries concise. One-line description, key points, usage syntax, expected result.

**Warning signs:** Entry takes more than 15-20 lines on screen.

### Pitfall 2: Inconsistent tone between files

**What goes wrong:** help.md uses imperatives, README.md uses conversational tone, GSD-STYLE.md uses descriptive tone — but if they mix, users are confused.

**Why it happens:** Each file has different purpose, easy to drift.

**How to avoid:**
- help.md: Reference style, imperative
- README.md: Conversational, user-facing
- GSD-STYLE.md: Descriptive, explanatory

**Warning signs:** Reading different files feels like different voices.

### Pitfall 3: Prescriptive rules in GSD-STYLE.md

**What goes wrong:** Telling users when they MUST use quick mode, instead of explaining what it is.

**Why it happens:** Pattern docs often drift into rules.

**How to avoid:** GSD-STYLE.md should describe WHAT quick mode is and HOW it works, not WHEN to use it (that's user's decision).

**Warning signs:** Language like "always use quick mode for..." or "never use quick mode when..."

### Pitfall 4: Documentation drift from implementation

**What goes wrong:** Documentation describes behavior that no longer matches the code.

**Why it happens:** Code changes, docs not updated.

**How to avoid:** Source documentation details from the actual command file (`commands/gsd/quick.md`).

**Warning signs:** Can't point to specific line in code that backs up documentation claim.

## Code Examples

### help.md Entry Format

```markdown
### Quick Mode

**`/gsd:quick`**
Execute small, ad-hoc tasks with GSD guarantees but skip optional agents.

Quick mode uses the same system with a shorter path:
- Spawns planner + executor (skips researcher, checker, verifier)
- Quick tasks live in `.planning/quick/` separate from planned phases
- Updates STATE.md tracking (not ROADMAP.md)

Use when you know exactly what to do and the task is small enough to not need research or verification.

Usage: `/gsd:quick`
Result: Creates `.planning/quick/NNN-slug/PLAN.md`, `.planning/quick/NNN-slug/SUMMARY.md`
```

### README.md Section Format

```markdown
### Quick Mode

```
/gsd:quick
```

**For ad-hoc tasks that don't need full planning.**

Quick mode gives you GSD guarantees (atomic commits, state tracking) with a faster path:

- **Same agents** — Planner + executor, same quality
- **Skips optional steps** — No research, no plan checker, no verifier
- **Separate tracking** — Lives in `.planning/quick/`, not phases

Use for: bug fixes, small features, config changes, one-off tasks.

```
/gsd:quick
> What do you want to do? "Add dark mode toggle to settings"
```

**Creates:** `.planning/quick/001-add-dark-mode-toggle/PLAN.md`, `SUMMARY.md`
```

### GSD-STYLE.md Pattern Format

```markdown
## Quick Mode Patterns

Quick mode provides GSD guarantees for ad-hoc tasks without full planning overhead.

### When to Use Quick Mode

**Quick mode:**
- Task is small and self-contained
- You know exactly what to do (no research needed)
- Task doesn't warrant full phase planning
- Mid-project fixes or small additions

**Full planning:**
- Task involves multiple subsystems
- You need to investigate approach first
- Task is part of a larger phase
- Task might have hidden complexity

[... additional subsections with structure and tracking info ...]
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| No quick mode | Quick mode v1.7.0 | 2026-01-19 | Users can now do ad-hoc tasks without full overhead |
| Ad-hoc tasks undocumented | Documented in help.md, README.md, GSD-STYLE.md | Phase 02 | Discoverability of quick mode feature |

**Deprecated/outdated:** None - this is new feature documentation.

## Open Questions

None - documentation patterns are well-established in this codebase. The Quick Mode feature is complete from Phase 01, and documentation simply needs to reflect it.

## Sources

### Primary (HIGH confidence)

- `/Users/dustinober/Projects/get-shit-done/commands/gsd/quick.md` — Source of truth for quick mode behavior (lines 1-287)
- `/Users/dustinober/Projects/get-shit-done/commands/gsd/help.md` — Template for command reference format (lines 1-399)
- `/Users/dustinober/Projects/get-shit-done/GSD-STYLE.md` — Existing style patterns, including Quick Mode section (lines 379-444)
- `/Users/dustinober/Projects/get-shit-done/.planning/phases/02-documentation/02-CONTEXT.md` — User decisions for this phase
- `/Users/dustinober/Projects/get-shit-done/.planning/STATE.md` — Project state and decision log

### Secondary (MEDIUM confidence)

- Git history (HEAD:README.md) — Current README structure shows existing quick mode documentation placement
- `/Users/dustinober/Projects/get-shit-done/CHANGELOG.md` — v1.7.0 entry confirms feature completion

### Tertiary (LOW confidence)

None - all findings verified against source files.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - Documentation files are the "stack", fully analyzed
- Architecture: HIGH - Existing patterns analyzed from actual files
- Pitfalls: HIGH - Derived from documentation best practices and existing file analysis

**Research date:** 2026-01-19
**Valid until:** 30 days (documentation patterns are stable)
