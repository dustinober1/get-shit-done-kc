# GSD Quick Mode

## What This Is

A fast-path command (`/gsd:quick`) for GSD that executes small tasks with the same guarantees (atomic commits, STATE.md tracking) but skips optional verification agents. Reduces agent spawns from 5-8 to 2 (planner + executor) for tasks where the user already knows what to do.

**Shipped:** v1.0 on 2026-01-19

## Core Value

Same guarantees, 50-70% fewer tokens for simple tasks.

## Requirements

### Validated

- ✓ `/gsd:quick` command executes end-to-end — v1.0
- ✓ Pre-flight validation (ROADMAP.md check) — v1.0
- ✓ Sequential numbering in `.planning/quick/` — v1.0
- ✓ Spawns gsd-planner (unchanged, skips researcher/checker) — v1.0
- ✓ Spawns gsd-executor for each plan — v1.0
- ✓ Parallel wave execution for multi-plan tasks — v1.0
- ✓ Commits only files it edits/creates — v1.0
- ✓ STATE.md "Quick Tasks Completed" table — v1.0
- ✓ STATE.md "Last activity" update — v1.0
- ✓ help.md Quick Mode section — v1.0
- ✓ README.md Quick Mode section — v1.0
- ✓ GSD-STYLE.md Quick Mode Patterns — v1.0

### Active

- [ ] `/gsd:resume-work` handles decimal phases (3.1, 3.2) — v2
- [ ] `/gsd:squash-quick` — future enhancement

### Out of Scope

- `--plan-only` flag — MVP always executes
- `--after N` flag — always inserts after current phase
- `--standalone` flag — requires active project, no exceptions
- Node.js helper scripts — Claude handles decimal parsing inline
- Git status warnings — commits only its own files anyway
- Planner modifications — planner unchanged, orchestrator skips agents
- gsd-verifier — verification skipped by design
- Requirements mapping — quick tasks are ad-hoc

## Current State

**Shipped:** v1.0 GSD Quick Mode (2026-01-19)

**Delivered:**
- `commands/gsd/quick.md` (286 lines) — Main command with inline orchestration
- Documentation in help.md, README.md, GSD-STYLE.md
- `.planning/quick/` directory pattern for quick tasks
- Sequential numbering (001, 002, 003...)

**Codebase stats:**
- 6,495 lines across GSD command files
- 8 files changed, 544 insertions, 70 deletions for v1.0

**Known issues:**
- Phase 01 has no formal PLAN.md/SUMMARY.md artifacts (implemented directly)

## Next Milestone Goals

To be defined via `/gsd:new-milestone`.

Potential areas:
- Resume support for decimal phases
- Additional quick task utilities
- Enhanced state tracking

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| No planner changes | Quick mode is orchestrator-level, not agent-level | SUCCESSFUL — Planner unchanged |
| Sequential numbering | Maintains full traceability and state integrity | SUCCESSFUL — .planning/quick/NNN-slug/ |
| No flags for MVP | Simplest possible interface | SUCCESSFUL — Single command |
| Parallel wave execution | Matches execute-phase behavior | SUCCESSFUL — Reuses executor |
| Quick Tasks table in STATE.md | Better tracking than just Last activity | SUCCESSFUL — New table format |
| Error if no ROADMAP | Maintains state integrity, no standalone mode | SUCCESSFUL — Pre-flight check |

---
*Last updated: 2026-01-19 after v1.0 milestone*
