# Feature Research: Kilo Code Skills for GSD Commands

**Domain:** Agent Skills Format (Kilo Code Agent Skills Specification)
**Researched:** 2025-01-19
**Confidence:** HIGH

## Executive Summary

This research documents the capabilities and features of Kilo Code's Agent Skills format to inform the conversion of 24 GSD (Get Shit Done) commands from Claude Code slash command format to Kilo Code skills format.

**Key Finding:** Kilo Code implements the open [Agent Skills specification](https://agentskills.io/specification) — a lightweight, interoperable format for extending AI agents. Skills are markdown-based with YAML frontmatter, supporting both generic (all modes) and mode-specific discovery patterns.

**Critical Differences from GSD Commands:**
1. **Invocation:** GSD commands are user-initiated via `/gsd:name` — Skills are AI-activated based on semantic description matching
2. **File Structure:** GSD uses flat `commands/gsd/*.md` — Skills require directory-based `skill-name/SKILL.md`
3. **Frontmatter:** Skills require strict `name`/`description` format with directory-name matching
4. **Mode Support:** Skills can be mode-specific (`skills-{mode}/`) or generic (all modes)
5. **Context Loading:** Skills load full instructions on-demand, not at startup

---

## Feature Landscape

### Table Stakes (Required for Skills)

Features that are mandatory for a valid Kilo Code skill. Missing these means the skill won't load.

| Feature | Why Required | Complexity | Notes |
|---------|--------------|------------|-------|
| **Directory structure** | Skills MUST be in a directory matching the skill name | LOW | `skill-name/SKILL.md` — flat files won't work |
| **SKILL.md filename** | Required filename for the skill definition | LOW | Case-sensitive, must be exactly `SKILL.md` |
| **`name` frontmatter** | Required identifier, max 64 chars, lowercase + hyphens only | LOW | Must match directory name exactly |
| **`description` frontmatter** | Required for semantic matching, max 1024 chars | MEDIUM | Critical for AI activation — describes WHEN to use |
| **Markdown body** | Instructions for the agent | LOW | No format restrictions, write what helps the agent |

### Optional Features (Skills Capabilities)

Features that add value but aren't required for basic functionality.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **`license` field** | Specify licensing for shared skills | LOW | Optional SPDX identifier or reference to LICENSE file |
| **`metadata` field** | Custom key-value pairs (author, version, etc.) | LOW | Arbitrary mapping for additional properties |
| **`compatibility` field** | Environment requirements (product, packages, network) | LOW | Max 500 chars, indicates required tools/access |
| **`allowed-tools` field** | Pre-approve specific tools for the skill | MEDIUM | Space-delimited list, **experimental** support |
| **Mode-specific directories** | Skills only load in specific modes (code, architect, etc.) | LOW | `skills-{mode-slug}/` naming pattern |
| **Project vs Global scope** | Override global skills per-project or vice versa | MEDIUM | Project skills take precedence over global |
| **scripts/ directory** | Bundle executable code with skill | MEDIUM | Self-contained or documented dependencies |
| **references/ directory** | Additional documentation loaded on-demand | LOW | Smaller files = less context usage |
| **assets/ directory** | Templates, images, data files | LOW | Static resources for templates/examples |
| **Symlinks** | Share skills across machines or from central repo | LOW | `name` must match symlink name, not target |

### Mode-Specific Features

Kilo Code has built-in modes that skills can target:

| Mode | Slug | Purpose | Directory Pattern |
|------|------|---------|-------------------|
| Code | `code` | Implementation, code generation | `skills-code/` |
| Architect | `architect` | System design, planning | `skills-architect/` |
| Debug | `debug` | Troubleshooting, fixing issues | `skills-debug/` |
| Ask | `ask` | Learning, exploration (read-only) | `skills-ask/` |
| Orchestrator | `orchestrator` | Coordination between modes | `skills-orchestrator/` |

**Generic skills** go in `skills/` and load in all modes.

---

## GSD Command Format (Source Format)

Current GSD commands use Claude Code's slash command format:

### Frontmatter Structure

```yaml
---
name: gsd:command-name
description: What this command does
argument-hint: "[optional] [args]"
agent: gsd-subagent-type (optional)
allowed-tools:
  - Read
  - Write
  - Bash
  - Task
  - AskUserQuestion
  - ...
---
```

### Body Structure

```markdown
<objective>
What this command achieves
</objective>

<execution_context>
@references/to/files.md
</execution_context>

<context>
Variables like $ARGUMENTS
@file.references
</context>

<process>
Step-by-step instructions
</process>

<output>
Files created
</output>

<success_criteria>
- [ ] Checklist items
</success_criteria>

<offer_next>
What to show user next
</offer_next>
```

### File Location

```
commands/gsd/
├── new-project.md
├── plan-phase.md
├── execute-phase.md
├── verify-work.md
├── progress.md
├── help.md
└── ... (24 total)
```

---

## Conversion Requirements

### Required Mappings (GSD → Kilo Skills)

| GSD Feature | Kilo Skills Equivalent | Conversion Notes |
|-------------|------------------------|------------------|
| `name: gsd:command-name` | `name: command-name` | Remove `gsd:` prefix, lowercase, hyphens only |
| `description` | `description` | Keep but enhance for semantic matching (WHEN to use) |
| `<objective>` | Markdown `## Objective` section | Convert XML tag to markdown heading |
| `<process>` | Markdown instructions | Convert to clear step-by-step markdown |
| `allowed-tools` | Not supported in skills | Remove — skills have full tool access (or use experimental `allowed-tools`) |
| `argument-hint` | Not directly supported | Document in description/instructions |
| `<success_criteria>` | Markdown `## Success Criteria` | Convert XML tag to markdown |
| `<offer_next>` | Not applicable | Skills don't orchestrate multi-command flows |
| `@file.references` | Relative paths from skill root | Update to `references/` or include inline |
| `<execution_context>` | Inline or references/ | Convert to skill body or separate files |

### Structural Changes Required

**1. Flat File → Directory Structure**

```
BEFORE: commands/gsd/new-project.md
AFTER:  skills/gsd-new-project/SKILL.md
        skills/gsd-new-project/
        ├── scripts/           (if needed)
        ├── references/        (templates, docs)
        └── assets/            (resources)
```

**2. Frontmatter Changes**

```yaml
# GSD Command (BEFORE)
---
name: gsd:new-project
description: Initialize a new project with deep context gathering
argument-hint: ""
allowed-tools: [Read, Write, Bash, Task]
---

# Kilo Skill (AFTER)
---
name: gsd-new-project
description: Initialize a new project through questioning -> research -> requirements -> roadmap. Use when starting a new project from scratch or adding a milestone to an existing project. Spawns parallel research agents for domain ecosystem analysis.
license: MIT
metadata:
  author: gsd-project
  version: 1.0.0
---
```

**3. Body Content Conversion**

- Remove XML-style tags (`<objective>`, `<process>`, etc.)
- Convert to markdown headings
- Add explicit "When to activate" guidance for AI matching
- Remove subagent spawning instructions (skills are invoked directly, don't spawn)
- Convert file references to relative paths from skill root

**4. Multi-Command Workflows**

GSD commands often orchestrate multi-agent workflows (e.g., `/gsd:new-project` spawns 4 researcher agents, then a synthesizer). Skills cannot do this orchestration.

**Conversion strategy:**
- Create separate skills for each agent role (researcher, planner, executor)
- User manually invokes the sequence
- OR: Create a "meta-skill" that describes the full workflow but requires user to run each step

---

## Feature Dependencies

```
[Directory Structure]
    └──requires──> [name matches directory]

[Semantic Description]
    └──requires──> [Clear WHEN-to-use guidance]
                   └──enhances──> [AI activation accuracy]

[Mode-Specific Skills]
    └──requires──> [Valid mode slug]
                   └──conflicts──> [Generic skill with same name]

[Project Override]
    └──requires──> [Global skill exists]
                   └──overrides──> [Global skill version]

[scripts/ references/ assets/]
    └──requires──> [SKILL.md exists]
                   └──enhances──> [Skill capabilities]
```

### Dependency Notes

- **Directory structure is foundational:** Without correct `skill-name/SKILL.md` layout, skill won't be discovered
- **Name matching is strict:** The `name` field MUST exactly match the parent directory name or validation fails
- **Description quality determines activation:** Poor descriptions mean skills won't be invoked when relevant
- **Mode slugs must be valid:** Using invalid mode slug means skills never load
- **Project scope takes precedence:** A project skill with same name completely overrides global version

---

## Mode-Specific Skills vs Generic Skills

### Generic Skills (All Modes)

**Location:** `~/.kilocode/skills/` or `.kilocode/skills/`

**Use when:**
- Skill applies to any development task
- No mode-specific tool restrictions needed
- General-purpose workflows

**Example:** `skills/gsd-progress/` — showing project status is relevant in any mode

### Mode-Specific Skills

**Location:** `~/.kilocode/skills-{mode}/` or `.kilocode/skills-{mode}/`

**Use when:**
- Skill requires specific tool permissions
- Workflow is mode-specific (e.g., architect mode for planning)
- Want to reduce clutter in other modes

**Mode Slug Mapping:**

| Mode | Slug | Directory | Example Skills |
|------|------|-----------|----------------|
| Code | `code` | `skills-code/` | Implementation, refactoring |
| Architect | `architect` | `skills-architect/` | Planning, system design |
| Debug | `debug` | `skills-debug/` | Troubleshooting, diagnosis |
| Ask | `ask` | `skills-ask/` | Learning, explanation |
| Orchestrator | `orchestrator` | `skills-orchestrator/` | Coordination (built-in) |

### Priority and Overrides

When skills with same name exist in multiple locations:

1. **Project mode-specific** → Highest priority
2. **Project generic** →
3. **Global mode-specific** →
4. **Global generic** → Lowest priority

**Example:** If `gsd-progress` exists in both:
- `~/.kilocode/skills/gsd-progress/` (global)
- `.kilocode/skills-code/gsd-progress/` (project, code-mode)

The project code-mode version wins when in Code mode on that project.

---

## Skill Discovery and Loading

### How Skills Are Discovered

**At Kilo Code startup:**
1. Scan global skill directories (`~/.kilocode/skills*/`)
2. Scan project skill directories (`.kilocode/skills*/`)
3. Load YAML frontmatter for all skills (name + description only)
4. Include relevant skills in system prompt based on active mode

**When mode changes:**
1. Reload skills list for the new mode
2. Include mode-specific skills in system prompt

**Reload required:**
- After adding/modifying skills
- `Cmd+Shift+P` → "Developer: Reload Window"

### How Skills Are Activated

**Two mechanisms:**

1. **Semantic matching (AI-decided):**
   - Agent analyzes current task
   - Matches against skill descriptions
   - Loads full SKILL.md when match found
   - Follows instructions

2. **Explicit invocation (user-initiated):**
   - User types `/skill-name {description}`
   - Forces skill activation
   - Useful when semantic matching misses

### Progressive Disclosure

Skills use efficient context loading:

1. **Startup (~100 tokens):** Only `name` + `description` for all skills
2. **Activation (<5000 tokens):** Full SKILL.md when skill is invoked
3. **On-demand (as needed):** scripts/, references/, assets/ only when referenced

---

## Differences from Claude Code Slash Commands

| Aspect | Claude Code Commands | Kilo Code Skills |
|--------|---------------------|------------------|
| **Invocation** | User types `/command-name` | AI activates based on description OR `/skill-name` |
| **File structure** | Flat `commands/*.md` files | Directory `skill-name/SKILL.md` |
| **Frontmatter** | Custom fields (allowed-tools, agent, etc.) | Spec-defined (name, description, optional fields) |
| **Tool restrictions** | `allowed-tools` limits what agent can use | Full tool access (experimental `allowed-tools` exists) |
| **Subagent spawning** | Commands spawn subagents via `Task()` | Skills don't spawn — they ARE the instructions |
| **Multi-command workflows** | Orchestrator commands spawn sequences | Each skill is independent |
| **Mode specificity** | Commands available in all modes | Skills can be mode-specific or generic |
| **Interoperability** | Claude Code specific | Cross-platform (Claude Desktop, Cursor, VS Code, Kilo) |
| **Validation** | None | Strict name matching, character limits |

### What Needs Conversion

| GSD Pattern | Kilo Skills Approach |
|-------------|---------------------|
| `agent: gsd-planner` spawning | Not supported — user must invoke planner skill directly |
| `Task()` subagent calls | Not supported — skills are instructions, not orchestrators |
| `allowed-tools` restrictions | Not supported — skills have full tool access |
| `<offer_next>` routing | Not applicable — skills complete and return |
| Multi-phase workflows | Split into separate skills, user runs sequence |
| AskUserQuestion flows | Supported via AskUserQuestion tool |

### What Maps Directly

| GSD Pattern | Kilo Skills Equivalent |
|-------------|------------------------|
| `<objective>` | `## Objective` heading |
| `<process>` | `## Process` or `## Instructions` |
| `<context>` | `## Context` heading or inline |
| `<success_criteria>` | `## Success Criteria` checklist |
| `@file.references` | `See [reference](references/file.md)` or inline |
| XML-style tags | Markdown headings/sections |

---

## Anti-Features (What to Avoid)

| Anti-Feature | Why Problematic | Alternative |
|--------------|-----------------|-------------|
| **Subagent spawning** | Skills don't support `Task()` calls | Create separate skills for each role, document sequence |
| **allowed-tools restrictions** | Not fully supported, experimental | Document required tools in description/instructions |
| **Orchestration logic** | Skills are instructions, not orchestrators | Let user control workflow, skill handles one task |
| **Name-directory mismatch** | Validation fails, skill won't load | Ensure `name:` exactly matches directory name |
| **Uppercase in name** | Validation error (lowercase only) | Use lowercase letters, numbers, hyphens only |
| **Vague descriptions** | AI won't activate skill appropriately | Include specific keywords, describe WHEN to use |
| **Huge SKILL.md files** | Exceeds context budget, slow loading | Keep <500 lines, move detail to references/ |
| **Deep nesting references** | Harder to follow, one-level recommended | Keep file references one level deep |
| **Hardcoded absolute paths** | Breaks when skills move/shared | Use relative paths from skill root |
| **Assuming Claude Code features** | Not cross-compatible | Stick to Agent Skills spec features |

---

## GSD Command Categories for Conversion

### Category 1: Simple Conversion (LOW complexity)

Commands that are self-contained instructions, no subagent spawning:

| Command | Description | Conversion Approach |
|---------|-------------|---------------------|
| `gsd:help` | Show command reference | Convert to skill, add "when to use: need help with GSD" |
| `gsd:progress` | Check project status | Convert to skill, add semantic keywords for status checks |
| `gsd:whats-new` | Show changelog | Simple conversion |
| `gsd:list-phase-assumptions` | Show planned approach | Direct conversion |
| `gsd:map-codebase` | Analyze existing code | May need mode-specific (architect) |
| `gsd:discuss-phase` | Gather phase context | Direct conversion |
| `gsd:pause-work` | Create handoff file | Simple conversion |
| `gsd:resume-work` | Restore session context | Simple conversion |
| `gsd:add-todo` | Capture task as todo | Simple conversion |
| `gsd:check-todos` | List and work on todos | Simple conversion |
| `gsd:debug` | Debug session with persistence | Simple conversion |

### Category 2: Moderate Conversion (MEDIUM complexity)

Commands with some orchestration or state management:

| Command | Description | Conversion Considerations |
|---------|-------------|---------------------------|
| `gsd:verify-work` | UAT with diagnosis routing | Split: verify skill + separate fix planning |
| `gsd:plan-phase` | Create plans with research | Split: research skill + planning skill |
| `gsd:execute-phase` | Execute plans with verification | Split: execute skill + verify skill |
| `gsd:research-phase` | Domain research | Direct conversion (orchestrates less) |
| `gsd:quick` | Fast task execution | Direct conversion |
| `gsd:add-phase` | Add phase to roadmap | Simple but needs ROADMAP.md understanding |
| `gsd:insert-phase` | Insert decimal phase | Simple but needs ROADMAP.md understanding |
| `gsd:remove-phase` | Delete and renumber | Simple but needs ROADMAP.md understanding |
| `gsd:plan-milestone-gaps` | Gap closure planning | Moderate complexity |
| `gsd:audit-milestone` | Verify milestone completion | Moderate complexity |
| `gsd:complete-milestone` | Archive and tag | Moderate complexity |

### Category 3: Complex Conversion (HIGH complexity)

Commands with heavy orchestration, parallel agent spawning:

| Command | Description | Conversion Challenges |
|---------|-------------|----------------------|
| `gsd:new-project` | Full project init with 4x parallel researchers | MAJOR: Spawns 4 parallel agents, needs breakdown |
| `gsd:new-milestone` | Milestone planning with research | Similar to new-project |

**Orchestration Challenge:** `gsd:new-project` spawns 4 parallel researcher agents (stack, features, architecture, pitfalls), then a synthesizer. Skills cannot do this.

**Conversion Options:**

1. **Separate skills:** Create `gsd-research-stack`, `gsd-research-features`, etc. — user runs each manually
2. **Meta-skill with instructions:** Describe the full workflow, user follows steps
3. **Hybrid:** Core skill + documented "run these 4 skills first" in description

---

## MVP Definition

### Launch With (v1) — Core GSD Skills

Minimum set to validate the skills approach:

- [ ] **`gsd-progress`** — Check status, route to next action
- [ ] **`gsd-discuss-phase`** — Gather phase context before planning
- [ ] **`gsd-plan-phase`** — Create execution plans
- [ ] **`gsd-execute-phase`** — Execute plans
- [ ] **`gsd-verify-work`** — UAT and gap identification
- [ ] **`gsd-help`** — Command reference

### Add After Validation (v1.x)

- [ ] Todo management skills (`gsd-add-todo`, `gsd-check-todos`)
- [ ] Debug skill (`gsd-debug`)
- [ ] Roadmap management skills (add/insert/remove-phase)
- [ ] Session management (`pause-work`, `resume-work`)

### Future Consideration (v2+)

- [ ] Complex orchestrator commands (`new-project`, `new-milestone`)
- [ ] May require Kilo Code to add orchestration features
- [ ] Or accept that some workflows require manual skill sequencing

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Basic skill format (name/description) | HIGH | LOW | P1 |
| Directory structure | HIGH | LOW | P1 |
| Core workflow skills (plan/execute/verify) | HIGH | MEDIUM | P1 |
| Semantic description quality | HIGH | MEDIUM | P1 |
| Mode-specific skills | MEDIUM | LOW | P2 |
| Project/global override | MEDIUM | MEDIUM | P2 |
| scripts/ directory | LOW | MEDIUM | P3 |
| references/ directory | MEDIUM | LOW | P2 |
| Orchestration workflows | HIGH | HIGH | P3 (blocked) |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

---

## Competitor Feature Analysis

| Feature | Claude Code Commands | Kilo Skills | Our Approach (GSD for Kilo) |
|---------|---------------------|-------------|---------------------------|
| User invocation | `/command` always | `/skill` or semantic matching | Support both styles |
| Tool restrictions | `allowed-tools` | Experimental `allowed-tools` | Document in description |
| Subagent spawning | `Task()` supported | Not supported | Separate skills per role |
| Cross-platform | Claude only | Multi-platform | Leverage Kilo's portability |
| Mode specificity | All commands | Mode-specific or generic | Use mode-specific for relevant skills |
| Validation | None | Strict (name matching, char limits) | Follow spec for reliability |

---

## Open Questions

1. **Orchestration patterns:** How should we handle complex workflows like `new-project` that spawn parallel agents? Kilo Code doesn't currently support this pattern.

2. **Tool restrictions:** Should we rely on experimental `allowed-tools` field or just document required tools in descriptions?

3. **Skill composition:** Can one skill reference another? (e.g., "first run the gsd-research-stack skill") — needs verification.

4. **State management:** GSD commands rely heavily on `.planning/STATE.md` and other project state. Should skills document this dependency or include inline?

---

## Sources

- [Skills | Kilo Code Docs](https://kilo.ai/docs/agent-behavior/skills) — Official Kilo Code skills documentation (HIGH confidence)
- [Specification - Agent Skills](https://agentskills.io/specification) — Complete Agent Skills format specification (HIGH confidence)
- [Custom Modes | Kilo Code Docs](https://kilo.ai/docs/agent-behavior/custom-modes) — Mode-specific behavior and directories (HIGH confidence)
- [Stop Re-Explaining Workflows with SKILLS.md](https://blog.kilo.ai/p/ai-agent-workflows-with-skills-md) — Skill usage patterns (MEDIUM confidence)
- [Building a Reusable Skills Ecosystem for AI Agents](https://addozhang.medium.com/agent-skills-deep-dive-building-a-reusable-skills-ecosystem-for-ai-agents-ccb1507b2c0f) — Ecosystem overview (MEDIUM confidence)
- GSD command files — Source analysis of 24 commands in `/Users/dustinober/Projects/get-shit-done/commands/gsd/` (HIGH confidence)

---

*Feature research for: Kilo Code Agent Skills Format (GSD conversion project)*
*Researched: 2025-01-19*
