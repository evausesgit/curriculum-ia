# Claude Code Skills — Team Learning Guide

> This guide explains how to create, configure, and share Skills in Claude Code — reusable instructions that Claude automatically applies to the right tasks at the right time.

**Original course:** [Introduction to Agent Skills](https://anthropic.skilljar.com/introduction-to-agent-skills) — Anthropic Academy (free)  
**Format:** 6 lessons · certificate

---

## Table of Contents

1. [Why Skills?](#1-why-skills)
2. [Anatomy of a Skill](#2-anatomy-of-a-skill)
3. [Creating Your First Skill](#3-creating-your-first-skill)
4. [Advanced Configuration](#4-advanced-configuration)
5. [Skills vs. Other Customization Tools](#5-skills-vs-other-customization-tools)
6. [Sharing Skills with Your Team](#6-sharing-skills-with-your-team)
7. [Diagnosing and Fixing Problems](#7-diagnosing-and-fixing-problems)

---

## 1. Why Skills?

### The problem they solve

Every time you explain your PR conventions to Claude, you're repeating yourself. Every code review, you re-describe how you want feedback structured. Every commit message, you remind Claude of the expected format.

Skills let you teach these things to Claude **once**. Claude then applies them automatically — no need to rewrite them.

### What sets Skills apart from other options

Claude Code offers several ways to customize its behavior. Here's how Skills fit in:

| Mechanism | Loading | Best for |
|-----------|---------|----------|
| `CLAUDE.md` | Every conversation | Permanent project standards |
| **Skills** | On demand, by context | Task-specific expertise |
| Slash commands | Explicit invocation | One-off actions triggered manually |
| Hooks | Events (save, commit...) | Deterministic automations |

The key advantage of Skills: they only consume context when relevant. Your PR review checklist doesn't need to be in memory when you're debugging a performance issue.

---

## 2. Anatomy of a Skill

### SKILL.md file structure

A Skill is a folder containing a `SKILL.md` file. This file has two parts separated by YAML frontmatter:

```
.claude/skills/skill-name/
└── SKILL.md
```

```markdown
---
name: skill-name
description: What the skill does and when Claude should use it.
---

Detailed instructions for Claude.
Everything after the frontmatter is the skill's content.
```

### Frontmatter fields

| Field | Required | Constraints | Role |
|-------|----------|-------------|------|
| `name` | ✅ | Lowercase, numbers, hyphens. Max 64 characters. | Skill identifier |
| `description` | ✅ | Max 1,024 characters | Trigger criterion — what Claude reads to decide if the skill is relevant |
| `allowed-tools` | No | List of tools | Restricts available tools when the skill is active |
| `model` | No | Model identifier | Specifies which Claude model to use |

### How Claude decides to use a skill

At startup, Claude scans skill directories but only loads the `name` and `description` — not the content. When you send a message, it semantically compares your request against all available descriptions. If a description matches, Claude asks for confirmation before loading the full content.

> **Practical implication:** a well-written description = a skill that triggers at the right time. A vague description = an invisible skill.

### Where Skills live

| Type | Path | Scope |
|------|------|-------|
| Personal | `~/.claude/skills/` (macOS/Linux) or `C:\Users\<user>\.claude\skills\` (Windows) | All your projects |
| Project | `.claude/skills/` at the repo root | Everyone on this repo |

---

## 3. Creating Your First Skill

### Example: a PR description skill

We'll create a personal skill that standardizes PR description writing. Personal = it applies across all your projects.

**Step 1 — Create the directory:**
```bash
mkdir -p ~/.claude/skills/pr-description
```

**Step 2 — Create the SKILL.md file:**
```markdown
---
name: pr-description
description: Writes pull request descriptions. Use when creating a PR,
  summarizing changes, or when asked for a pull request description.
---

To write a PR description:

1. Run `git diff main...HEAD` to see all changes on the branch
2. Write the description in this format:

## What
One sentence explaining what this PR does.

## Why
Short context on why this change is needed.

## Changes
- Bullet points of specific changes made
- Group related changes together
- Mention deleted or renamed files
```

**Step 3 — Test:**
Restart your Claude Code session (Skills are loaded at startup), then say: *"write a PR description for my changes"*. Claude will confirm it's using your Skill and produce the same format every time.

### Priority when names conflict

When two Skills share the same name, the priority order is:

```
Enterprise > Personal > Project > Plugins
```

If your organization has an Enterprise `code-review` skill, your personal `code-review` skill will be ignored. Solution: use more specific names (`frontend-code-review`, `api-code-review`).

### Updating or removing a Skill

- **Update**: edit the `SKILL.md` file
- **Remove**: delete the entire folder
- **Important**: restart Claude Code after any change

---

## 4. Advanced Configuration

### Restricting tools with `allowed-tools`

By default, an active Skill imposes no restrictions on available tools. You can change this to create read-only Skills — useful for codebase exploration, onboarding, or security-sensitive workflows:

```yaml
---
name: codebase-explorer
description: Helps understand the architecture and how the system works.
  Use for onboarding or architecture questions.
allowed-tools: Read, Grep, Glob, Bash
model: sonnet
---
```

When this Skill is active, Claude cannot modify files — even if you explicitly ask it to.

### Progressive disclosure: structuring large Skills

A Skill's content is loaded entirely into context when activated. For complex Skills, loading 2,000 lines at once would be inefficient.

The solution: keep the essentials in `SKILL.md` and spread details across supporting files, loaded only when needed.

**Recommended structure:**
```
.claude/skills/my-skill/
├── SKILL.md              # Core instructions (< 500 lines)
├── references/           # Detailed documentation
│   └── arch-guide.md
├── scripts/              # Executable scripts
│   └── validate.sh
└── assets/               # Templates, data files
```

**In SKILL.md, explicitly state when to load supporting files:**
```markdown
For system architecture questions, read `references/arch-guide.md`.
For all other requests, use only the instructions below.
```

Claude only loads `arch-guide.md` when the question warrants it.

> **Rule of thumb:** if your `SKILL.md` exceeds 500 lines, some content should be moved to a reference file.

### Running scripts without consuming context

Scripts referenced in a Skill can execute without their content being loaded into context — only the execution result is visible to Claude. Tell Claude to **run** the script, not **read** it.

Particularly useful for:
- Environment validation
- Data transformations that need to be reproducible
- Operations better handled as tested code than generated code

---

## 5. Skills vs. Other Customization Tools

### Quick decision guide

```
Should the rule apply to EVERY conversation?
├── Yes → CLAUDE.md
└── No → Does it apply to specific tasks?
    ├── Yes → Skill
    └── Should it trigger on every occurrence of an event?
        ├── Yes → Hook
        └── Do you need an isolated execution context?
            ├── Yes → Subagent
            └── Do you need an external tool?
                └── Yes → MCP
```

### Detailed comparison

**CLAUDE.md vs Skills**

`CLAUDE.md` is always in memory — every conversation starts with that context. A Skill is only loaded when relevant.

Concrete example: *"always use TypeScript strict mode"* → `CLAUDE.md`. *"PR review checklist"* → Skill (no need to load it when debugging a performance issue).

**Skills vs Subagents**

A Skill adds knowledge to your current conversation. A subagent creates a new isolated execution thread with its own context.

Use a subagent when you want to delegate a complete task and receive only the result. Use a Skill when you want to enrich how Claude handles your current request.

**Skills vs Hooks**

Hooks trigger on events (a file save, a tool call). Skills trigger on intent (what you're asking for).

Use a hook for what must always happen automatically. Use a Skill for what Claude should know how to do when asked.

---

## 6. Sharing Skills with Your Team

Three sharing levels, depending on desired scope.

### Level 1: via Git repository

The simplest method. Project Skills live in `.claude/skills/`. By committing this folder, everyone who clones the repo gets them automatically — and updates propagate via `git pull`.

**Best for:**
- Team coding standards
- Project-specific workflows
- Skills that reference your codebase structure

### Level 2: via plugins

Plugins allow distributing Skills beyond a single repo, via marketplaces. Each user installs the plugin in their Claude Code.

**Best for:** Skills useful to the community, not tightly coupled to a specific project.

### Level 3: via Enterprise Managed Settings

Administrators can deploy Skills to the entire organization. These Skills have **absolute priority** — they override personal, project, and plugin Skills with the same name.

Managed settings also allow restricting plugin installation sources:
```json
"strictKnownMarketplaces": [
  { "source": "github", "repo": "my-org/approved-plugins" },
  { "source": "npm", "package": "@my-org/compliance-plugins" }
]
```

**Best for:** mandatory standards, security requirements, compliance workflows.

### Skills and subagents: an important caveat

Subagents do **not** automatically see available Skills — they start with an empty context. Additionally:

- Built-in agents (Explorer, Plan, Verify) **cannot** access Skills at all
- Only **custom** subagents can use Skills, and only if you list them explicitly

To create a subagent with Skills, use `/agents` → "Create new agent". The generated frontmatter looks like:

```yaml
---
name: frontend-reviewer
description: "Use for frontend code review, accessibility, and performance."
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch
model: sonnet
color: blue
skills: accessibility-audit, performance-check
---
```

This pattern is particularly useful for specialized subagents: a frontend reviewer with accessibility Skills, a backend reviewer with security Skills.

---

## 7. Diagnosing and Fixing Problems

### First tool to reach for: the validator

Before any manual debugging, run the Skills validator. It detects structural problems (malformed frontmatter, incorrect path, etc.) before you waste time looking elsewhere.

Recommended installation via `uv`. Run it from your Skills folder or from anywhere by passing the path as an argument.

### Quick diagnostic checklist

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Skill doesn't trigger | Description too vague or misaligned with your requests | Add phrasings you actually use |
| Skill doesn't appear in list | Wrong file structure | Check that `SKILL.md` is in a named subfolder, not at the root |
| Wrong skill used | Descriptions too similar between skills | Make descriptions more distinct and specific |
| Personal skill ignored | Conflict with a higher-priority skill | Rename your skill with a more specific name |
| Plugin skills missing | Corrupted cache | Clear cache, restart Claude Code, reinstall plugin |
| Runtime error | Missing dependency, permissions, path | See below |

### Skill doesn't trigger

Claude uses semantic matching — your request must have intent that overlaps with the description. If it doesn't trigger:

1. Compare your description with how you actually phrase your requests
2. Add alternative phrasings: *"audit performance"*, *"why is this slow?"*, *"optimize this code"*
3. Test several variants — if one doesn't trigger, add those words to the description

### Skill isn't loaded

Strict structural requirements:
- The file must be named exactly `SKILL.md` — `SKILL` in uppercase, `.md` in lowercase
- It must be inside a named subfolder (e.g. `~/.claude/skills/my-skill/SKILL.md`), not directly in `~/.claude/skills/`

Run `claude --debug` to see loading errors. Look for messages mentioning your skill name.

### Runtime errors

Three common causes:

- **Missing dependencies** — if your Skill uses external packages, they must be installed. Add dependency info to your description so Claude knows what's needed
- **Script permissions** — referenced scripts must be executable: `chmod +x my-script.sh`
- **Path separators** — use forward slashes (`/`) everywhere, even on Windows
