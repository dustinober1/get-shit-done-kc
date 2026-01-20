# Stack Research

**Domain:** npm packages that install to Kilo Code's skills directory
**Researched:** 2026-01-19
**Confidence:** HIGH

## Executive Summary

Kilo Code supports the **Agent Skills** open standard as of January 2026. An npm package targeting Kilo Code should:
1. Use Node.js built-in modules only (no runtime dependencies)
2. Install skills to `~/.kilocode/skills/[name]/` (global) or `.kilocode/skills/[name]/` (local)
3. Convert Claude Code command format to Agent Skills format (SKILL.md with YAML frontmatter)
4. Use a `bin/install.js` script consistent with the existing `get-shit-done-cc` pattern

The conversion strategy is straightforward at install time: transform each `commands/gsd/*.md` file into an Agent Skills-compatible `SKILL.md` by wrapping the command content in the standard YAML frontmatter format.

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Node.js | >=16.7.0 | Runtime for installer script | Matches existing get-shit-done-cc engine requirement; built-in modules sufficient |
| npm | Current | Package distribution | Standard Node package manager; `bin` field creates CLI command |
| Agent Skills Format | 1.0 | Skill specification | Open standard supported by Kilo Code (as of Jan 2026), Claude Code, GitHub Copilot |

### Package Structure (for get-shit-done-kc)

```
get-shit-done-kc/
├── package.json              # Package manifest
├── bin/
│   └── install.js            # CLI installer (Node.js script)
├── commands/                 # Source: Claude Code commands (existing)
│   └── gsd/
│       ├── new-project.md
│       ├── execute-phase.md
│       └── ...
└── skills-src/               # NEW: Kilo Code skill templates (optional)
    └── gsd/
        └── SKILL.md.template # Template for single unified skill
```

### Installation Targets

| Location | Purpose | When Used |
|----------|---------|-----------|
| `~/.kilocode/skills/gsd/` | Global skills directory | Available across all projects |
| `.kilocode/skills/gsd/` | Local skills directory | Project-specific installation |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| (None) | N/A | Zero runtime dependencies | Use Node.js built-in modules only (fs, path, os, readline) |

## Installation

```bash
# No dependencies to install
# The package is self-contained in the installer script
```

**package.json for get-shit-done-kc:**

```json
{
  "name": "get-shit-done-kc",
  "version": "1.0.0",
  "description": "GSD for Kilo Code - AI coding agent workflow system",
  "bin": {
    "get-shit-done-kc": "bin/install.js"
  },
  "files": [
    "bin",
    "commands",
    "skills-src"
  ],
  "keywords": [
    "kilo-code",
    "ai",
    "agent-skills",
    "workflow",
    "development"
  ],
  "author": "TACHES",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/glittercowboy/get-shit-done.git"
  },
  "engines": {
    "node": ">=16.7.0"
  }
}
```

## Kilo Skills Format Requirements

### Agent Skills Specification (HIGH Confidence)

Kilo Code uses the Agent Skills open standard. A skill is a folder containing a `SKILL.md` file:

**File Structure:**
```
gsd/
└── SKILL.md    # Required: YAML frontmatter + instructions
```

**SKILL.md Format:**
```markdown
---
name: gsd-new-project
description: Initialize a new project with deep context gathering and PROJECT.md
---

# GSD: New Project

<instructions>
...
</instructions>
```

**Required Frontmatter Fields:**
- `name`: Short identifier (lowercase, hyphens for spaces)
- `description`: When to use this skill (discovery prompt)

**Markdown Body:**
- No specific restrictions on structure
- Contains the actual instructions, examples, and guidelines
- Can reference external files with `@` syntax

### Skill Discovery and Loading

Kilo Code uses **progressive disclosure**:
1. **Discovery**: At startup, only `name` and `description` are loaded
2. **Activation**: When a task matches the description, full `SKILL.md` is read into context
3. **Execution**: Agent follows instructions, optionally loading referenced files

### Directory Placement

| Scope | Path | Priority |
|-------|------|----------|
| Global | `~/.kilocode/skills/` | Available across all projects |
| Local | `.kilocode/skills/` | Project-specific, takes precedence |

## Conversion Strategy: Claude Code to Kilo Skills

### Claude Code Command Format (Existing)

```markdown
---
name: gsd:new-project
description: Initialize a new project with deep context gathering
allowed-tools:
  - Read
  - Bash
  - Write
---

<objective>
...
</objective>

<process>
...
</process>
```

### Kilo Skills Format (Target)

```markdown
---
name: gsd-new-project
description: Initialize a new project with deep context gathering and PROJECT.md
---

# GSD: New Project

<objective>
...
</objective>

<process>
...
</process>
```

### Conversion Differences

| Aspect | Claude Code | Kilo Code | Action |
|--------|-------------|-----------|--------|
| Name format | `gsd:new-project` (colon) | `gsd-new-project` (hyphen) | Replace `:` with `-` |
| Frontmatter | `allowed-tools` list | Not supported | Strip this field |
| Invocation | `/gsd:new-project` | Mention by name or auto-discovery | Update docs |

### Installer Conversion Logic

```javascript
// Pseudocode for bin/install.js
function convertClaudeCommandToKiloSkill(srcCmdPath, destSkillPath) {
  const content = fs.readFileSync(srcCmdPath, 'utf8');

  // Parse YAML frontmatter
  const frontmatterMatch = content.match(/^---\n(.*?)\n---/s);
  let frontmatter = frontmatterMatch[1];
  const body = content.slice(frontmatterMatch[0].length);

  // Transform name: gsd:new-project -> gsd-new-project
  frontmatter = frontmatter.replace(/name:\s*gsd:/, 'name: gsd-');

  // Remove allowed-tools (not supported in Kilo)
  frontmatter = frontmatter.replace(/allowed-tools:\s*\n((?:  -.*\n)*)/, '');

  // Build SKILL.md
  const skillContent = `---\n${frontmatter}---\n\n${body}`;

  fs.writeFileSync(destSkillPath, skillContent);
}
```

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Convert at install time | Publish separate Kilo skills repo | When you want Kilo-specific customizations per skill |
| Single skill folder `gsd/` | Individual folders per command | When each command should be discoverable separately |
| Node.js installer | Shell script installer | When targeting non-Node users (not recommended - inconsistent) |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Runtime dependencies (fs-extra, etc.) | Adds install friction, increases attack surface | Node.js built-in modules (fs, path, os) |
| Shell scripts for cross-platform logic | Platform-specific edge cases on Windows | Pure Node.js for all file operations |
| YAML parsing libraries | Not needed for simple frontmatter transforms | String manipulation / regex |
| Post-install script (`scripts.postinstall`) | Not reliably run in all npm scenarios | Use `bin` CLI command pattern |
| Chaining `npx` calls | Complex, error-prone | Single `npx get-shit-done-kc` command |

## Stack Patterns by Variant

**If installing globally:**
- Use `~/.kilocode/skills/gsd/` as destination
- Skills available across all projects
- Recommend for most users

**If installing locally:**
- Use `.kilocode/skills/gsd/` in current directory
- Skills scoped to this project only
- Useful for per-project customization

**If user has `CLAUDE_CONFIG_DIR` set:**
- This env var does NOT apply to Kilo Code
- Kilo Code always uses `~/.kilocode/` for global skills
- Document this difference clearly

## Path Resolution Across Platforms

| Platform | Home Directory | Global Skills Path |
|----------|----------------|-------------------|
| Linux | `$HOME` | `~/.kilocode/skills/` |
| macOS | `$HOME` | `~/.kilocode/skills/` |
| Windows | `%USERPROFILE%` | `C:\Users\{username}\.kilocode\skills\` |

**Node.js portable pattern:**
```javascript
const os = require('os');
const path = require('path');

// Works on all platforms
const globalSkillsDir = path.join(os.homedir(), '.kilocode', 'skills');
```

**Tilde expansion:**
```javascript
// Shell doesn't expand ~ in node, need manual handling
function expandTilde(filePath) {
  if (filePath && filePath.startsWith('~/')) {
    return path.join(os.homedir(), filePath.slice(2));
  }
  return filePath;
}
```

## Version Compatibility

| Package | Compatible With | Notes |
|-----------|-----------------|-------|
| get-shit-done-kc | Kilo Code >=4.79 | Agent Skills support added Jan 2026 |
| get-shit-done-kc | Node.js >=16.7.0 | Matches existing get-shit-done-cc requirement |
| Agent Skills format | Claude Code, GitHub Copilot | Cross-platform compatibility |

## Package Publishing Workflow

1. Update `package.json` version
2. Run `npm pack` to create tarball
3. Test install: `npm install -g get-shit-done-kc-1.0.0.tgz`
4. Publish: `npm publish`
5. User install: `npx get-shit-done-kc`

## Sources

- [Agent Skills Specification - What are skills?](https://agentskills.io/what-are-skills) - HIGH confidence: Official Agent Skills documentation
- [GitHub - anthropics/skills](https://github.com/anthropics/skills) - HIGH confidence: Official Anthropic implementation
- [Kilo Code Skills Discussion #4604](https://github.com/Kilo-Org/kilocode/discussions/4604) - HIGH confidence: Kilo team confirmed Agent Skills support (Jan 2026)
- [npm Docs - package.json bin field](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#bin) - HIGH confidence: Official npm documentation
- [Existing get-shit-done-cc installer](/Users/dustinober/Projects/get-shit-done/bin/install.js) - HIGH confidence: Reference implementation
- [Kilo Code Custom Rules Documentation](https://kilo.ai/docs/agent-behavior/custom-rules) - HIGH confidence: Official Kilo docs on directory structure

---
*Stack research for: GSD for Kilo Code*
*Researched: 2026-01-19*
