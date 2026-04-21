# Introduction to Agent Skills

> Learn how to build, configure, and share Skills in Claude Code — reusable markdown instructions that Claude automatically applies to the right tasks at the right time.

**Source:** https://anthropic.skilljar.com/introduction-to-agent-skills  
**Format:** 6 lessons · Certificate of completion  
**Prix:** Gratuit

---

## About this course

In this course, you'll learn how to stop repeating yourself and start teaching Claude once. You'll discover what Skills are and how they differ from other Claude Code customization options like CLAUDE.md, hooks, and subagents. You'll create your first Skill from scratch — writing the SKILL.md frontmatter, crafting effective descriptions that reliably trigger matching, and organizing your skill directory with progressive disclosure to keep context windows efficient. You'll also explore advanced configuration options like restricting tool access with `allowed-tools` and using scripts that execute without consuming context.

Beyond building individual Skills, you'll learn how to share them with your team by committing them to a repository, distribute them more broadly through plugins, and deploy them organization-wide using enterprise managed settings. You'll see how to wire Skills into custom subagents for isolated, expert task delegation, and you'll walk through a complete troubleshooting guide for diagnosing issues — from skills that won't trigger to priority conflicts and runtime errors.

---

## Lesson 1 — What are skills?

*Estimated time: 15 minutes*

**By the end of this lesson you'll be able to:**
- Define what Claude Code skills are and how they work
- Explain where skills live (personal vs. project directories)
- Distinguish between skills, CLAUDE.md, and slash commands
- Identify scenarios where skills are the right customization tool

---

Every time you explain your team's coding standards to Claude, you're repeating yourself. Every PR review, you re-describe how you want feedback structured. Every commit message, you remind Claude of your preferred format. Skills fix this.

A skill is a markdown file that teaches Claude how to do something once. Claude then applies that knowledge automatically whenever it's relevant.

### What Skills Are

Skills are folders of instructions and resources that Claude Code can discover and use to handle tasks more accurately. Each skill lives in a `SKILL.md` file with a name and description in its frontmatter.

The description is how Claude decides whether to use the skill. When you ask Claude to review a PR, it matches your request against available skill descriptions and finds the relevant one.

Here's what a skill's frontmatter looks like:

```yaml
---
name: pr-review
description: Reviews pull requests for code quality. Use when reviewing PRs or checking code changes.
---
```

Below the frontmatter, you write the actual instructions — your review checklist, formatting preferences, or whatever Claude needs to know for that task.

### Where Skills Live

| Location | Path | Scope |
|----------|------|-------|
| Personal | `~/.claude/skills` (macOS/Linux) or `C:/Users/<user>/.claude/skills` (Windows) | All your projects |
| Project | `.claude/skills` inside a repository | Anyone who clones the repo |

Project skills get committed to version control alongside your code, so the whole team shares them.

### Skills vs. CLAUDE.md vs. Slash Commands

- **CLAUDE.md** loads into every conversation — use it for always-on project standards (e.g. always use TypeScript strict mode).
- **Skills** load on demand when they match your request. Claude only loads the name and description initially, so they don't fill up your context window. Your PR review checklist doesn't need to be in context when you're debugging.
- **Slash commands** require you to explicitly type them. Skills don't — Claude applies them when it recognizes the situation.

### When to Use Skills

Skills work best for specialized knowledge that applies to specific tasks:
- Code review standards your team follows
- Commit message formats you prefer
- Brand guidelines for your organization
- Documentation templates for specific types of docs
- Debugging checklists for particular frameworks

**Rule of thumb:** if you find yourself explaining the same thing to Claude repeatedly, that's a skill waiting to be written.

---

## Lesson 2 — Creating your first skill

*Estimated time: 20 minutes*

**By the end of this lesson you'll be able to:**
- Create a skill from scratch with proper frontmatter structure
- Test and verify that a skill loads correctly in Claude Code
- Explain how Claude Code matches incoming requests to available skills
- Describe the skill priority hierarchy (Enterprise, Personal, Project, Plugins)

---

### Creating a Skill

We'll build a personal skill that teaches Claude how to write PR descriptions in a consistent format. Since it's a personal skill, it lives in your home directory and works across all your projects.

First, create a directory for your skill inside the skills folder:

```bash
mkdir -p ~/.claude/skills/pr-description
```

Then create a `SKILL.md` file inside that directory:

```markdown
---
name: pr-description
description: Writes pull request descriptions. Use when creating a PR, writing a PR, or when the user asks to summarize changes for a pull request.
---

When writing a PR description:

1. Run `git diff main...HEAD` to see all changes on this branch
2. Write a description following this format:

## What
One sentence explaining what this PR does.

## Why
Brief context on why this change is needed

## Changes
- Bullet points of specific changes made
- Group related changes together
- Mention any files deleted or renamed
```

The **name** identifies your skill. The **description** tells Claude when to use it — this is the matching criteria. Everything after the second set of dashes is the instructions Claude follows when the skill is activated.

### Testing Your Skill

Claude Code loads skills at startup, so **restart your session** after creating one. You can verify it's available by checking the available skills list.

To test it, make some changes on a branch and say something like *"write a PR description for my changes."* Claude will indicate it's using the PR description skill, check your diff, and write a description following your template — same format every time.

### How Skill Matching Works

When Claude Code starts, it scans four locations for skills but only loads the **name and description** — not the full content. When you send a request, Claude compares your message against the descriptions of all available skills using semantic matching.

Once a match is found, Claude asks you to confirm loading the skill. After you confirm, Claude reads the complete `SKILL.md` file and follows its instructions.

### Skill Priority

When two skills have the same name, this priority order applies:

| Priority | Source |
|----------|--------|
| 1 (highest) | Enterprise (managed settings) |
| 2 | Personal (`~/.claude/skills`) |
| 3 | Project (`.claude/skills` in repo) |
| 4 (lowest) | Plugins |

To avoid conflicts, use descriptive names: `frontend-review` instead of just `review`.

### Updating and Removing Skills

- To update a skill: edit its `SKILL.md` file
- To remove a skill: delete its directory
- Always **restart Claude Code** after any changes

---

## Lesson 3 — Configuration and multi-file skills

*Estimated time: 20 minutes*

**By the end of this lesson you'll be able to:**
- Configure advanced skill metadata fields including `allowed-tools` and `model`
- Write effective skill descriptions that reliably trigger on the right requests
- Use `allowed-tools` to restrict what Claude can do when a skill is active
- Organize complex skills using progressive disclosure and multi-file structures

---

### Skill Metadata Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | ✅ | Lowercase letters, numbers, hyphens only. Max 64 chars. Should match directory name. |
| `description` | ✅ | How Claude decides when to use the skill. Max 1,024 chars. Most important field. |
| `allowed-tools` | Optional | Restricts which tools Claude can use when the skill is active. |
| `model` | Optional | Specifies which Claude model to use for the skill. |

### Writing Effective Descriptions

A good description answers two questions:
1. **What does the skill do?**
2. **When should Claude use it?**

If your skill isn't triggering when you expect it to, add more keywords that match how you actually phrase your requests.

### Restricting Tools with `allowed-tools`

Use `allowed-tools` when you want a skill that can only read files, not modify them — useful for security-sensitive workflows or read-only tasks:

```yaml
---
name: codebase-onboarding
description: Helps new developers understand how the system works.
allowed-tools: Read, Grep, Glob, Bash
model: sonnet
---
```

When this skill is active, Claude can only use those tools. If you omit `allowed-tools` entirely, the skill doesn't restrict anything.

### Progressive Disclosure

Keep essential instructions in `SKILL.md` and put detailed reference material in separate files that Claude reads only when needed.

**Recommended directory structure:**
```
.claude/skills/my-skill/
├── SKILL.md          # Core instructions (keep under 500 lines)
├── scripts/          # Executable code
├── references/       # Additional documentation
└── assets/           # Images, templates, or other data files
```

In `SKILL.md`, link to supporting files with clear instructions about when to load them. For example: *"Read `references/architecture-guide.md` only when asked about system design."*

**Rule of thumb:** keep `SKILL.md` under 500 lines. If you're exceeding that, split content into separate reference files.

### Using Scripts Efficiently

Scripts in your skill directory can run **without loading their contents into context**. The script executes and only the output consumes tokens. Tell Claude to *run* the script, not *read* it.

This is useful for:
- Environment validation
- Data transformations that need to be consistent
- Operations that are more reliable as tested code than generated code

---

## Lesson 4 — Skills vs. other Claude Code features

*Estimated time: 15 minutes*

**By the end of this lesson you'll be able to:**
- Compare skills to CLAUDE.md, subagents, hooks, and MCP servers
- Choose the right Claude Code customization feature for a given use case
- Design a complementary setup that combines multiple features effectively

---

### Decision Guide

| Feature | Trigger | Best for |
|---------|---------|----------|
| **CLAUDE.md** | Every conversation | Always-on project standards, constraints, framework preferences |
| **Skills** | On demand (semantic match) | Task-specific expertise, procedures relevant only sometimes |
| **Subagents** | Explicit delegation | Isolated execution contexts, delegated work with different tool access |
| **Hooks** | Events (file save, tool call) | Auto-formatting, validation, automated side effects |
| **MCP servers** | Tool calls | External tools and integrations |

### CLAUDE.md vs Skills

**Use CLAUDE.md for:**
- Project-wide standards that always apply
- Constraints like "never modify the database schema"
- Framework preferences and coding style

**Use Skills for:**
- Task-specific expertise
- Knowledge that's only relevant sometimes
- Detailed procedures that would clutter every conversation

### Skills vs Subagents

Skills **add knowledge to your current conversation** — the instructions join the existing context.

Subagents **run in a separate context** — they receive a task, work on it independently, and return results.

**Use Subagents when:**
- You want to delegate a task to a separate execution context
- You need different tool access than the main conversation

**Use Skills when:**
- You want to enhance Claude's knowledge for the current task
- The expertise applies throughout a conversation

### Skills vs Hooks

Hooks fire on **events** (e.g. every time Claude saves a file). Skills activate based on **what you're asking**.

**Use Hooks for:** operations that should run on every file save, validation before specific tool calls.

**Use Skills for:** knowledge that informs how Claude handles requests.

### Putting It All Together

A typical setup:
- **CLAUDE.md** — always-on project standards
- **Skills** — task-specific expertise that loads on demand
- **Hooks** — automated operations triggered by events
- **Subagents** — isolated execution contexts for delegated work
- **MCP servers** — external tools and integrations

---

## Lesson 5 — Sharing skills

*Estimated time: 20 minutes*

**By the end of this lesson you'll be able to:**
- Share skills with your team by committing them to a Git repository
- Distribute skills across projects through plugins and marketplaces
- Deploy skills organization-wide using enterprise managed settings
- Configure custom subagents to use specific skills

---

### Method 1: Committing Skills to Your Repository

Place skills in `.claude/skills`. Anyone who clones the repo gets them automatically — no extra installation needed. When you push updates, everyone gets them on the next pull.

Best for:
- Team coding standards
- Project-specific workflows
- Skills that reference your codebase structure

### Method 2: Distributing Skills Through Plugins

Create a plugin with a `skills/` directory following the same file structure as `.claude/`. After publishing to a marketplace, other users can install it into Claude Code.

Best when your skills aren't too project-specific and can be useful to the broader community.

### Method 3: Enterprise Deployment Through Managed Settings

Administrators can deploy skills organization-wide. Enterprise skills take the **highest priority** — they override personal, project, and plugin skills with the same name.

The managed settings file supports `strictKnownMarketplaces` to control where plugins can be installed from:

```json
"strictKnownMarketplaces": [
  {
    "source": "github",
    "repo": "acme-corp/approved-plugins"
  },
  {
    "source": "npm",
    "package": "@acme-corp/compliance-plugins"
  }
]
```

Best for mandatory standards, security requirements, and compliance workflows.

### Skills and Subagents

**Important:** subagents don't automatically see your skills. When you delegate a task to a subagent, it starts with a fresh, clean context.

- **Built-in agents** (Explorer, Plan, Verify) **cannot** access skills at all
- **Custom subagents** can use skills, but only when explicitly listed in the agent's frontmatter

To create a custom subagent with skills, use `/agents` → "Create new agent". The generated frontmatter looks like:

```yaml
---
name: frontend-security-accessibility-reviewer
description: "Use this agent when you need to review frontend code for accessibility..."
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch, Skill...
model: sonnet
color: blue
skills: accessibility-audit, performance-check
---
```

This pattern works well when:
- Different subagents need different skills (frontend reviewer vs. backend reviewer)
- You want to enforce standards in delegated work without relying on prompts

---

## Lesson 6 — Troubleshooting skills

*Estimated time: 15 minutes*

**By the end of this lesson you'll be able to:**
- Use the skills validator to catch structural issues before debugging
- Diagnose and fix common skill triggering and loading problems
- Resolve skill priority conflicts between enterprise, personal, project, and plugin skills
- Debug runtime errors including missing dependencies, permissions, and path issues

---

### Quick Troubleshooting Checklist

| Symptom | Fix |
|---------|-----|
| **Not triggering** | Improve your description, add trigger phrases that match how you actually phrase requests |
| **Not loading** | Check path, file name (`SKILL.md` exactly), and YAML syntax |
| **Wrong skill used** | Make descriptions more distinct from each other |
| **Being shadowed** | Check the priority hierarchy and rename if needed |
| **Plugin skills missing** | Clear cache and reinstall |
| **Runtime failure** | Check dependencies, permissions (`chmod +x`), and path separators (use `/` everywhere) |

### Step 1: Use the Skills Validator

Run the agent skills verifier command first. It catches structural problems before you spend time debugging other things. (Installation steps vary by OS; `uv` is the easiest method.)

### Skill Doesn't Trigger

The cause is almost always the **description**. Claude uses semantic matching, so your request needs to overlap with the description's meaning.

- Check your description against how you're actually phrasing requests
- Add trigger phrases users would actually say
- Test with variations like *"help me profile this,"* *"why is this slow?",* *"make this faster"*

### Skill Doesn't Load

Check these structural requirements:
- The `SKILL.md` file must be **inside a named directory**, not at the skills root
- The file name must be exactly `SKILL.md` — all caps on "SKILL", lowercase "md"

Run `claude --debug` to see loading errors. Look for messages mentioning your skill name.

### Wrong Skill Gets Used

Your descriptions are probably too similar. Make them more distinct and specific.

### Skill Priority Conflicts

If your personal skill is being ignored, an enterprise or higher-priority skill might have the same name. Options:
- Rename your skill to something more distinct (usually the easier path)
- Talk to your admin about the enterprise skill

### Plugin Skills Not Appearing

Clear the cache, restart Claude Code, and reinstall. If skills still don't appear, the plugin structure might be wrong — use the validator tool.

### Runtime Errors

Common causes:
- **Missing dependencies:** add dependency info to your skill description so Claude knows what's needed
- **Permission issues:** run `chmod +x` on any scripts your skill references
- **Path separators:** use forward slashes everywhere, even on Windows
