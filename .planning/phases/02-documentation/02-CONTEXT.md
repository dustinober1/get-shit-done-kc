# Phase 02: Documentation - Context

**Gathered:** 2026-01-19
**Status:** Ready for planning

<domain>
## Phase Boundary

Document Quick mode (`/gsd:quick` command) across three files: help.md (command reference), README.md (overview for new users), and GSD-STYLE.md (patterns for GSD power users).

</domain>

<decisions>
## Implementation Decisions

### help.md entry
- Placement: Claude's discretion (you decide where it fits best)
- Include side-by-side comparison: Quick vs. Regular mode showing when to use each
- Include full example: Show actual `/gsd:quick "task name"` invocation with output
- Guarantee detail level: Claude's discretion (appropriate for command reference)

### README.md section
- Placement: Claude's discretion (place where it flows best)
- Prominence: Full section with `## Quick Mode` heading (not subsection, not brief callout)
- When-to-use guidance: Claude's discretion (include if it helps decision-making)

### GSD-STYLE.md patterns
- Content scope: Claude's discretion (document what's most useful for GSD power users)
- Prescriptiveness: Descriptive only — explain what quick mode does, let reader decide when to use it (not rules or heuristics)
- Code examples: Claude's discretion (include if they illustrate patterns clearly)

### Claude's Discretion
- help.md section placement
- help.md guarantee detail level
- README.md section placement
- README.md when-to-use guidance inclusion
- GSD-STYLE.md content scope
- GSD-STYLE.md code example inclusion

</decisions>

<specifics>
## Specific Ideas

- help.md: Side-by-side comparison table (Quick vs. Regular) is important
- README.md: Full dedicated section, not buried
- GSD-STYLE.md: Descriptive tone, not prescriptive rules

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 02-documentation*
*Context gathered: 2026-01-19*
