# Claude Code — Complete Learning Path

> This guide aggregates the full Claude Code curriculum into a single progressive learning journey: from foundations to advanced automation techniques.

**Sources:** [Claude Code 101](https://anthropic.skilljar.com/claude-code-101) · [Introduction to Agent Skills](https://anthropic.skilljar.com/introduction-to-agent-skills) · [Best practices](https://code.claude.com/docs/en/best-practices) — Anthropic Academy & Official documentation

---

## Table of Contents

**Part 1 — Understanding Claude Code**
1. [What Claude Code Actually Is](#1-what-claude-code-actually-is)
2. [The Central Constraint: the Context Window](#2-the-central-constraint-the-context-window)

**Part 2 — Getting Started**
3. [Installation by Environment](#3-installation-by-environment)
4. [Slash Commands](#4-slash-commands)
5. [Writing Your First Prompt](#5-writing-your-first-prompt)

**Part 3 — Core Workflow**
6. [The Explore → Plan → Code → Commit Workflow](#6-the-explore--plan--code--commit-workflow)
7. [Writing Good Prompts](#7-writing-good-prompts)
8. [Managing Context Actively](#8-managing-context-actively)

**Part 4 — Verification and Quality**
9. [Give Claude a Way to Verify Its Work](#9-give-claude-a-way-to-verify-its-work)
10. [Code Review with Claude](#10-code-review-with-claude)

**Part 5 — Customizing Claude Code**
11. [CLAUDE.md: Persistent Memory](#11-claudemd-persistent-memory)
12. [Permission Modes](#12-permission-modes)
13. [Hooks](#13-hooks)
14. [Subagents](#14-subagents)
15. [MCP — Model Context Protocol](#15-mcp--model-context-protocol)
16. [Skills](#16-skills)

**Part 6 — Advanced Mastery**
17. [Let Claude Interview You](#17-let-claude-interview-you)
18. [Managing Your Session Like a Pro](#18-managing-your-session-like-a-pro)
19. [Automate and Scale](#19-automate-and-scale)
20. [Common Failure Patterns](#20-common-failure-patterns)

[Global Reference](#global-reference)

---

## Part 1 — Understanding Claude Code

## 1. What Claude Code Actually Is

### The difference from a chat assistant

Most developers have already used Claude or ChatGPT by copy-pasting code back and forth. Claude Code works differently: it has **direct access** to your filesystem, your terminal, and your repository. It doesn't need you to show it the code — it reads files itself, edits them, and runs commands.

That's what makes it an **agent**, not just an assistant.

### What is an agent?

An AI agent is a program that perceives its environment and takes actions to reach a goal. In practice, Claude Code can:

- **Read and understand your codebase** — explore files, trace a bug, understand an architecture
- **Edit files across your project** — refactor a function and update every file that references it
- **Run terminal commands** — execute tests, install dependencies, read logs, and use the output to decide what to do next
- **Search the web** — look up documentation or API references on the fly

### The agentic loop

Every time you send a message, Claude Code follows this loop:

```
Your prompt
    ↓
Context gathering (reading files, web search...)
    ↓
Action (file edit, command execution...)
    ↓
Verification (do the results meet the goal?)
    ↓
Yes → waits for your next message
No  → loops back and tries again
```

You can interrupt or steer Claude at any point in this loop.

### Three things to keep in mind

**The context window is its working memory.** Claude can hold a lot in mind at once, but not your entire project. It explores your codebase strategically to find what it needs.

**It asks for your approval before acting.** By default, Claude Code waits for your confirmation before editing a file or running a command. You stay in control.

**It can make mistakes.** Like any tool, it can misinterpret a request or introduce a bug. Staying in the loop lets you catch errors early.

---

## 2. The Central Constraint: the Context Window

Almost every best practice in this guide flows from a single constraint: **Claude's context window fills up fast, and performance degrades as it fills.**

Everything Claude "sees" during a session — your messages, every file it reads, every command output — occupies space in its context window. A single debugging session can consume tens of thousands of tokens. As the window fills, Claude may start forgetting earlier instructions or making more mistakes.

**The context window is the most important resource to manage.**

Keep this in mind: every technique you'll learn here — precise prompts, subagents, on-demand skills, `/compact` and `/clear` — aims, directly or indirectly, to preserve that space for what actually matters.

---

## Part 2 — Getting Started

## 3. Installation by Environment

Claude Code runs in several environments. Pick the one that fits your workflow.

### Terminal (macOS / Linux / WSL)

```bash
# Install via curl (supports auto-updates)
curl -fsSL https://claude.ai/install.sh | sh

# Launch Claude Code in your project
cd my-project
claude
```

On first launch: choose a color theme, sign in with your Claude account (Pro, Max, or Enterprise) or an API key.

> Claude Code has access to the directory you launch it from and all its subdirectories.

### Windows

Available via PowerShell, CMD, or `winget`. Note: `winget` and Homebrew don't support auto-updates — prefer `curl` when possible.

### VS Code

1. Extensions panel → search **"Claude Code"** (Anthropic extension, blue verification badge)
2. Install → restart VS Code if prompted
3. `Ctrl/Cmd + Shift + P` → "Claude Code Open in New Tab"

The experience is nearly identical to the terminal. A setting lets you disable the UI and use the built-in terminal directly.

### JetBrains

Install the **Claude Code** plugin from the JetBrains Marketplace. After restarting, the Claude logo appears in the sidebar and opens a terminal panel integrated with your editor.

### Claude Desktop

Open Claude Desktop → enable the **"Code"** toggle at the top. Lets you work in a specific folder with configurable permissions, and run Claude in the background.

### Web (claude.ai/code)

Access via `claude.ai/code` or the sidebar in Claude.ai. Same experience as Desktop, but limited to GitHub repositories.

### Which environment to choose?

| Need | Recommended |
|------|-------------|
| Latest features first | Terminal |
| Stay inside your editor | VS Code / JetBrains |
| Run Claude in the background | Desktop |
| Work on a GitHub repo remotely | Web |

---

## 4. Slash Commands

Slash commands are built-in instructions you type directly in the Claude Code prompt. They don't get sent to the model as conversation messages — they trigger specific behaviors in the Claude Code application itself.

Think of them as the control panel of your session: context management, state inspection, tool configuration, workflow launching — all without leaving the terminal.

Type `/` then Tab to see all available commands.

### Session & context

| Command | What it does | When to use it |
|---------|-------------|----------------|
| `/context` | Shows context window usage: total size, breakdown by category, visual chart | Before a long task, or when responses feel less sharp |
| `/compact` | Summarizes and compresses the current context to free up space | Mid-feature, when approaching the context limit |
| `/clear` | Wipes the entire context — fresh session with no memory | Starting a new, unrelated task |
| `/cost` | Shows the token cost of the current session | Tracking usage or debugging unexpectedly large context |
| `/rewind` | Opens the rewind menu to restore a previous state | After a wrong turn, or to return to a checkpoint |
| `/btw` | Asks a quick question without adding it to context | Checking a detail without growing the session |
| `/rename` | Renames the current session | To find it easily later |

### Project setup

| Command | What it does | When to use it |
|---------|-------------|----------------|
| `/init` | Analyzes your project and generates a `CLAUDE.md` file | First time setting up Claude Code on a project |

### Tools & integrations

| Command | What it does | When to use it |
|---------|-------------|----------------|
| `/mcp` | Lists connected MCP servers, their status, lets you enable/disable them | Managing external integrations |
| `/agents` | Opens the subagent manager: list, create, or edit subagents | Setting up or reviewing agents |
| `/hooks` | Opens the hook configurator | Adding or reviewing deterministic guardrails |
| `/permissions` | Manages command allowlists | Reducing repetitive permission prompts |

### Git & code

| Command | What it does | When to use it |
|---------|-------------|----------------|
| `/commit-push-pr` | Commits changes, pushes the branch, and opens a PR — in one step | End of a feature when you're ready to ship |

### Help & diagnostics

| Command | What it does | When to use it |
|---------|-------------|----------------|
| `/help` | Lists all available commands | When you forget a command name |
| `/doctor` | Diagnoses your Claude Code installation (config, auth, connectivity) | Something feels broken |

### CLI flags (outside the session)

| Flag | What it does |
|------|-------------|
| `claude --continue` | Picks up the most recent session |
| `claude --resume` | Chooses from a list of saved sessions |
| `claude --from-pr <number>` | Resumes a session linked to a specific PR |
| `claude -p "prompt"` | Runs Claude non-interactively (CI, scripts) |
| `claude --debug` | Starts a session with verbose debug output |
| `claude mcp add <name>` | Adds a new MCP server to your configuration |

---

## 5. Writing Your First Prompt

### Choosing your interaction mode

`Shift + Tab` cycles through available modes:

| Mode | Behavior |
|------|----------|
| **Approval (default)** | Claude asks before each file edit or command |
| **Auto-accept** | File edits happen automatically; still asks for commands |
| **Plan Mode** | Uses only read-only tools to explore and plan before any action |

> **Team tip:** start in Approval mode while getting familiar with what Claude does, then switch to Auto-accept for routine tasks.

### Plan Mode: essential for complex tasks

Plan Mode is the right choice for multi-step implementations. In this mode, Claude uses only read-only tools to explore your code and ask clarifying questions, then submits a detailed plan for your approval before writing a single line.

**Example prompt in Plan Mode:**
```
My app needs a dark mode system.
Create a toggle in the header that switches between light and dark mode
across the entire app, consistent with my existing color palette.
```

Claude will explore your CSS/Tailwind files, ask questions if needed, then walk you through a step-by-step plan before any code is written.

---

## Part 3 — Core Workflow

## 6. The Explore → Plan → Code → Commit Workflow

This is the core workflow for teams to adopt. It prevents the main trap: asking for code immediately without establishing context, which leads to costly corrections mid-session.

### Explore

Before touching any code, give Claude the context it needs. In Plan Mode, Claude explores your codebase read-only. You can also trigger exploration explicitly to get an architecture summary without any intention to make changes.

```
I need to add WebP conversion to our image upload pipeline.
Tell me where in the pipeline it should happen, what dependencies
would be needed, and how you'd approach the implementation.
```

### Plan

Claude reads relevant files, does research if needed, and submits an action plan. This is the **best time to course-correct** — before any code is written. Ask questions, request revisions on specific points. A well-validated plan leads to a clean implementation.

### Code

Once the plan is approved, Claude works through the steps. Tips for this phase:

- **Explicit success criterion** — tell Claude what "correct" looks like (green tests, expected behavior)
- **Test suite** — give Claude a test suite to continuously validate against; it can also write one for you
- **Right tools** — for web UIs, the Claude in Chrome extension lets Claude control a browser tab and test visually
- **Memorize corrections** — if Claude keeps making the same mistake, ask it to save the fix to `CLAUDE.md`

### Commit

Before pushing your code:

1. **Run a subagent review** — fresh eyes, without the bias of the coding session (see section 10)
2. **Ask Claude to generate a commit message** in your team's style

---

## 7. Writing Good Prompts

### Prompting patterns

The level of precision in your prompt directly determines result quality and how much context gets consumed.

| Strategy | Weak | Strong |
|----------|------|--------|
| **Scope the task** | *"add tests for foo.py"* | *"write a test for foo.py covering the edge case where the user is logged out. avoid mocks."* |
| **Point to sources** | *"why does ExecutionFactory have such a weird api?"* | *"look through ExecutionFactory's git history and summarize how its api came to be"* |
| **Reference patterns** | *"add a calendar widget"* | *"look at HotDogWidget.php to understand our widget pattern, then implement a calendar widget that lets the user pick a month and paginate by year. no new libraries."* |
| **Describe the symptom** | *"fix the login bug"* | *"users report login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test, then fix it"* |
| **Verification criterion** | *"implement validateEmail"* | *"write validateEmail. test cases: user@example.com → true, invalid → false, user@.com → false. run the tests after."* |

### Ways to provide rich context

- **`@filename`** — reference a file directly; Claude reads it before responding
- **Paste images** — copy/paste or drag screenshots, designs, or diagrams into the prompt
- **Give URLs** — paste documentation links; use `/permissions` to allowlist frequently-used domains
- **Pipe data** — `cat error.log | claude` sends file contents directly as input
- **Let Claude fetch context** — tell Claude to use Bash, MCP tools, or file reads to pull what it needs

### Use CLI tools

Claude is highly effective with CLI tools like `gh` (GitHub), `aws`, `gcloud`, and `sentry-cli`. These are more context-efficient than their MCP equivalents and Claude already knows how to use them.

```
Use gh to list open PRs assigned to me, then summarize each one.
```

For tools Claude doesn't know: `Use 'foo-cli --help' to learn it, then use it to do X.`

---

## 8. Managing Context Actively

### The essential commands

| Command | Effect | When to use |
|---------|--------|-------------|
| `/compact` | Compacts history into a summary | Mid-feature, when approaching the limit |
| `/clear` | Wipes everything, fresh start | Between distinct features |
| `/context` | Shows size, categories, breakdown | To diagnose what's consuming context |

> **Rule of thumb:** `/compact` during a feature, `/clear` between features. What you want Claude to remember across sessions → put it in `CLAUDE.md`.

### Strategies to save context

**Be specific in your prompts.** A vague prompt forces Claude to explore more broadly to guess your intent — this consumes far more context than a detailed prompt would.

**Disable unused MCP servers.** Every connected MCP server loads its tool definitions into context, even when you're not using it. Check with `/mcp`.

**Use subagents for investigation.** When Claude researches a codebase it reads lots of files, all of which consume your context. Subagents run in separate context windows and report back summaries.

```
Use subagents to investigate how our authentication system handles token
refresh, and whether we have any existing OAuth utilities I should reuse.
```

---

## Part 4 — Verification and Quality

## 9. Give Claude a Way to Verify Its Work

> Give Claude a check it can run: tests, a build, a screenshot to compare. It's the difference between a session you watch and one you walk away from.

Claude stops when the work *looks* done. Without a check it can run, you become the verification loop — every mistake waits for you to notice it. The check is anything that returns a pass or fail Claude can read: a test suite, a build exit code, a linter, a script that diffs output against a fixture.

### Levels of verification

| Level | Mechanism | Setup | Best for |
|-------|-----------|-------|----------|
| In one prompt | Ask Claude to run the check and iterate | None | Any task |
| Across a session | `/goal` condition — a separate evaluator re-checks after every turn | Low | Long tasks |
| Deterministic gate | Stop hook — blocks the turn from ending until the check passes | Medium | Unattended runs |
| Second opinion | Verification subagent — a fresh model reviews the result | Medium | Critical code |

### Ask for evidence, not assertions

Have Claude show you the test output, the command it ran, or a screenshot — rather than just saying "done". Reviewing evidence is faster than re-running the verification yourself, and it works for sessions you weren't watching.

---

## 10. Code Review with Claude

### Subagent review: guaranteed fresh perspective

Before pushing a PR, delegate the review to a subagent. The key advantage: the subagent starts with a clean context, without the bias built up during the coding session.

**Recommended configuration:**
- Read-only tools only — a reviewer flags issues, it doesn't fix them
- Version the configuration in your repo so the whole team uses the same reviewer

### Writer / Reviewer pattern

A fresh context improves code review — Claude won't be biased toward code it just wrote.

| Session A (Writer) | Session B (Reviewer) |
|--------------------|----------------------|
| `Implement a rate limiter for our API endpoints` | |
| | `Review the rate limiter in @src/middleware/rateLimiter.ts. Look for edge cases, race conditions, and consistency with our existing middleware.` |
| `Here's the review feedback: [output]. Address these issues.` | |

You can do the same with tests: one Claude writes tests, another writes code to pass them.

### Adversarial review

Before treating a task as done, have a subagent review the result in a fresh context.

```
Use a subagent to review the rate limiter diff against PLAN.md. Check that
every requirement is implemented, the listed edge cases have tests, and
nothing outside the task's scope changed. Report gaps, not style preferences.
```

> A reviewer prompted to find gaps will usually report some, even when the work is sound. Tell it to flag only gaps that affect correctness or stated requirements.

### Commit, push, and PR in one command

The `/commit-push-pr` skill chains commit, push, and PR creation in a single step. If you have a Slack MCP server configured with channels listed in `CLAUDE.md`, it can also automatically post the PR link to your team channel.

### Resuming work on an existing PR

```bash
claude --from-pr <PR_NUMBER>
```

Useful for addressing review comments or fixing a broken build.

---

## Part 5 — Customizing Claude Code

## 11. CLAUDE.md: Persistent Memory

`CLAUDE.md` is a Markdown file at the root of your project. Claude reads it automatically at the start of every conversation. Think of it as an **onboarding document for Claude** — it prevents rediscovering the same things every session.

### Recommended minimal structure

```markdown
# Project

Next.js 15 app using App Router, Tailwind CSS, and Drizzle ORM.

# Commands
- Dev: `pnpm dev`
- Tests: `pnpm test`
- Lint: `pnpm lint`

# Conventions
- 2-space indentation
- Named exports over default exports
- API routes go in `app/api/`
- Prefer Server Actions over API routes where possible
```

### What to include — and what to cut

`CLAUDE.md` is loaded every session. Keep it short — a bloated file causes Claude to ignore the rules buried in the noise.

| ✅ Include | ❌ Exclude |
|-----------|----------|
| Bash commands Claude can't guess | Anything Claude can infer from reading code |
| Code style rules that differ from defaults | Standard language conventions |
| Test runner and testing instructions | Detailed API documentation (link instead) |
| Branch naming and PR conventions | Information that changes frequently |
| Architectural decisions specific to your project | Long explanations or tutorials |
| Developer environment quirks (required env vars) | File-by-file descriptions of the codebase |

**Rule of thumb:** for each line, ask *"Would removing this cause Claude to make mistakes?"* If not, cut it.

### Memory file hierarchy

| File | Location | Scope |
|------|----------|-------|
| Project `CLAUDE.md` | Repo root | Whole team (commit to git) |
| `CLAUDE.local.md` | Repo root | Personal only (add to `.gitignore`) |
| Personal `CLAUDE.md` | `~/.claude/CLAUDE.md` | You only, all your projects |
| Subdirectory `CLAUDE.md` | Loaded on demand when reading files there | Local context |

### Practical tips

**Let the content emerge.** Start without a `CLAUDE.md` and observe where you keep correcting Claude. Those corrections are worth memorializing. Use `/init` to have Claude generate a first version automatically.

**Reference existing documentation:**
```markdown
## Architecture
For more context: @docs/architecture.md
```

**Memorize recurring corrections.** If you find yourself repeating the same instruction, ask Claude: *"Save this rule to the project CLAUDE.md."*

**If a rule keeps getting ignored** despite being in `CLAUDE.md`, the file is probably too long. Convert critical rules to hooks for deterministic enforcement.

---

## 12. Permission Modes

By default, Claude requests approval before every file write or command. After the tenth approval, you're no longer really reviewing — you're just clicking through.

| Mode | How | When to use |
|------|-----|-------------|
| **Approval (default)** | Asks before each action | New tasks, unfamiliar codebases |
| **Auto-accept** | Edits files without asking | Well-understood routine tasks |
| **Auto mode** | Classifier blocks risky actions; routine work proceeds | Trust the direction, don't want to click every step |
| **Allowlists** | Permit specific safe commands like `npm run lint` or `git commit` | Known safe operations you run constantly |
| **Sandboxing** | OS-level isolation restricts filesystem and network access | Unattended runs with external or untrusted inputs |

Configure via `/permissions` or directly in `.claude/settings.json`.

---

## 13. Hooks

Hooks run scripts automatically at specific points in Claude's workflow. Unlike `CLAUDE.md` instructions which are advisory, hooks are **deterministic** and guarantee the action happens.

### Available events

| Event | Trigger |
|-------|---------|
| `PreToolUse` | Before a tool call executes |
| `PostToolUse` | After a tool call completes |
| `UserPromptSubmit` | When you submit a prompt, before Claude processes it |
| `Stop` | When Claude finishes responding |
| `Notification` | When Claude sends a notification |

### Typical team use cases

- **Auto-formatting**: `PostToolUse` on `Edit|MultiEdit|Write` → run Prettier, `gofmt`, etc.
- **Audit logging**: record every command Claude executes
- **Guardrails**: block edits to prod config, `rm -rf`, direct commits to `main`
- **Automatic verification**: Stop hook that runs tests and blocks the turn if they fail
- **Notifications**: Slack or desktop alert when a long task completes

### Blocking with PreToolUse

A `PreToolUse` hook can block an action before it executes. It receives the tool name and its parameters as JSON on `stdin`:

| Exit code | Behavior |
|-----------|----------|
| `0` | Action proceeds normally |
| `2` | Action blocked — `stderr` message is fed back to Claude so it understands why |
| Other | Non-blocking error visible to you only |

### Sharing hooks with the team

Commit `.claude/settings.json` to your repo. The whole team automatically gets the same guardrails. Use `CLAUDE_PROJECT_DIR` to reference scripts stored in your project.

> **Principle:** what must happen every time without exception → a hook. What is a preference → `CLAUDE.md`.

---

## 14. Subagents

A subagent is a Claude instance that runs in parallel with its own isolated context. It receives a task, processes it, and returns only its result — without polluting your main context.

### Why it matters

When Claude researches a codebase, it reads dozens of files — all of which consume context. If you only need the final answer, all that exploratory work is noise. A subagent does the same exploration in its own bubble and returns just the conclusion.

### Creating a subagent

```
/agents → "Create new agent"
```

You define: scope (personal or project), purpose, available tools, and when Claude should call it automatically. Subagents are Markdown files with YAML frontmatter — versionable in your repo.

**Example — security review subagent:**
```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
model: opus
---
You are a senior security engineer. Review code for:
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication and authorization flaws
- Secrets or credentials in code

Provide specific line references and suggested fixes.
```

### Invoking a subagent

```
Use a subagent to investigate how our authentication system handles token refresh.
Use a subagent to review this code for edge cases.
```

---

## 15. MCP — Model Context Protocol

MCP is an open standard that connects Claude Code to external tools and data sources — Linear, Slack, GitHub, dependency documentation, databases...

### Adding an MCP server

```bash
claude mcp add <server-name>
```

Two types: **HTTP** (remote services) and **Stdio** (local processes).

### Server scoping

| Scope | Config file | Sharing |
|-------|-------------|---------|
| Local | Personal settings | You only |
| User | Global settings | All your projects |
| Project | `.mcp.json` in repo | Whole team automatically |

> To share MCPs with the team: commit `.mcp.json` at the repo root. Everyone gets the same servers on the next `git pull`.

### Watch your context

Every connected MCP server loads its tool definitions into context, even when unused. If a tool has a CLI equivalent (`gh`, `aws`...), the CLI is more context-efficient. Disable unused servers with `/mcp`.

---

## 16. Skills

### The problem they solve

Every time you explain your PR conventions to Claude, you're repeating yourself. Skills let you teach these things to Claude **once**. Claude then applies them automatically — no need to rewrite them.

### How Skills compare to other options

| Mechanism | Loading | Best for |
|-----------|---------|----------|
| `CLAUDE.md` | Every conversation | Permanent project standards |
| **Skills** | On demand, by context | Task-specific expertise |
| Slash commands | Explicit invocation | One-off actions triggered manually |
| Hooks | Events (save, commit...) | Deterministic automations |

The key advantage of Skills: they only consume context when relevant. Your PR review checklist doesn't need to be in memory when you're debugging a performance issue.

### Anatomy of a Skill

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

| Field | Required | Role |
|-------|----------|------|
| `name` | ✅ | Identifier (lowercase, hyphens, max 64 chars) |
| `description` | ✅ | Trigger criterion — what Claude reads to decide if the skill is relevant |
| `allowed-tools` | No | Restricts available tools when the skill is active |
| `model` | No | Specifies which Claude model to use |

> **Key rule:** a well-written description = a skill that triggers at the right time. A vague description = an invisible skill.

### Creating your first Skill

**Example — PR description skill:**

```bash
mkdir -p ~/.claude/skills/pr-description
```

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
```

Restart your Claude Code session (Skills are loaded at startup), then say: *"write a PR description for my changes"*.

### Where Skills live

| Type | Path | Scope |
|------|------|-------|
| Personal | `~/.claude/skills/` | All your projects |
| Project | `.claude/skills/` at repo root | Everyone on this repo |

### Advanced configuration

**Restricting tools with `allowed-tools`:**
```yaml
---
name: codebase-explorer
description: Helps understand the architecture. Use for onboarding questions.
allowed-tools: Read, Grep, Glob, Bash
model: sonnet
---
```

When this Skill is active, Claude cannot modify files.

**Progressive disclosure for large Skills:**
```
.claude/skills/my-skill/
├── SKILL.md              # Core instructions (< 500 lines)
├── references/           # Loaded on demand
└── scripts/              # Executable scripts
```

In `SKILL.md`, explicitly state when to load supporting files: `"For architecture questions, read references/arch-guide.md."`

### Sharing Skills with your team

| Level | Method | Best for |
|-------|--------|----------|
| Git repo | Commit `.claude/skills/` | Team coding standards, project workflows |
| Plugins | Claude Code marketplace | Skills useful to the community |
| Enterprise | Managed Settings | Mandatory standards, compliance requirements |

> Enterprise Skills have **absolute priority** — they override personal, project, and plugin Skills with the same name.

### Skills and subagents

Subagents do **not** automatically see available Skills — they start with an empty context. To create a subagent with Skills:

```yaml
---
name: frontend-reviewer
description: "Use for frontend code review, accessibility, and performance."
tools: Bash, Glob, Grep, Read
model: sonnet
skills: accessibility-audit, performance-check
---
```

### Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Skill doesn't trigger | Description too vague | Add phrasings you actually use |
| Skill doesn't appear in list | Wrong file structure | Check that `SKILL.md` is in a named subfolder |
| Wrong skill used | Descriptions too similar | Make descriptions more distinct |
| Personal skill ignored | Priority conflict | Rename with a more specific name |

Run `claude --debug` to see loading errors.

---

## Part 6 — Advanced Mastery

## 17. Let Claude Interview You

> For larger features, have Claude interview you first — it asks about things you might not have considered yet.

Instead of writing a full spec yourself, start with a minimal description and ask Claude to extract requirements through structured questions.

```
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
Don't ask obvious questions — dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Once the spec is complete, **start a fresh session** to implement it. The new session has clean context focused entirely on implementation, and you have a written reference to check against.

**What makes a good spec:** it names the files and interfaces involved, states what is explicitly out of scope, and ends with an end-to-end verification step that proves the feature works.

---

## 18. Managing Your Session Like a Pro

### Course-correct early and often

The best results come from tight feedback loops. Correct Claude as soon as you notice it going off track — waiting is never faster.

| Action | How | When |
|--------|-----|------|
| Stop mid-action | `Esc` | Claude is doing the wrong thing |
| Undo and redirect | `Esc + Esc` or `/rewind` → restore | Wrong approach, want to try differently |
| Revert changes | "Undo that" | Keep the conversation, roll back the code |
| Start fresh | `/clear` | New unrelated task, or after two failed corrections |

> **Rule:** if you've corrected Claude more than twice on the same issue, run `/clear` and write a better prompt that incorporates what you learned. A clean session with a better prompt almost always outperforms a long session with accumulated corrections.

### Checkpoints and rewinding

Every prompt you send creates a checkpoint. Claude snapshots files before each change.

- `Esc + Esc` or `/rewind` — opens the rewind menu
- Options: restore conversation only, restore code only, restore both, or summarize from a message
- Checkpoints persist across sessions — close your terminal and rewind later

This means you can tell Claude to try something risky. If it doesn't work, rewind and try a different approach.

> Checkpoints only track changes made *by Claude*, not external processes. This is not a replacement for git.

### Resuming sessions

```bash
claude --continue        # Pick up the most recent session
claude --resume          # Choose from a list of saved sessions
```

Use `/rename` to give sessions descriptive names like `oauth-migration` so you can find them later. Treat sessions like branches: each workstream gets its own persistent context.

### Side questions with `/btw`

For quick questions you don't need in context, use `/btw`. The answer appears in a dismissible overlay and never enters conversation history.

```
/btw what does the --no-ff flag do in git merge?
```

---

## 19. Automate and Scale

### Non-interactive mode

Use `claude -p "prompt"` to run Claude without a session — in CI, pre-commit hooks, or scripts.

```bash
# One-off query
claude -p "Explain what this project does"

# Structured output for scripts
claude -p "List all API endpoints" --output-format json

# Streaming for real-time processing
claude -p "Analyze this log file" --output-format stream-json --verbose
```

Use `--allowedTools` to restrict what Claude can do in automated runs:

```bash
claude -p "Fix all lint errors" --allowedTools "Edit,Bash(npm run lint)"
```

### Run multiple sessions in parallel

| Approach | How | Best for |
|----------|-----|----------|
| Worktrees | Separate CLI sessions in isolated git checkouts | Parallel features without edit collisions |
| Desktop app | Multiple local sessions, each in its own worktree | Visual management of parallel workstreams |
| Web (claude.ai/code) | Sessions on cloud infrastructure in isolated VMs | Remote work on GitHub repos |
| Agent teams | Automated coordination with shared tasks and a team lead | Complex multi-agent workflows |

### Fan-out across files

For large migrations or analyses, distribute work across many parallel Claude invocations.

**Step 1 — Generate a task list:**
```
list all 200 Python files that need migrating from requests to httpx
```

**Step 2 — Write a loop:**
```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from requests to httpx. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

**Step 3 — Test on 2-3 files, then scale.** Refine your prompt based on what goes wrong first.

### Run autonomously with auto mode

For uninterrupted execution with background safety checks:

```bash
claude --permission-mode auto -p "fix all lint errors"
```

A classifier blocks scope escalation, unknown infrastructure, and hostile-content-driven actions while letting routine work proceed without prompts.

---

## 20. Common Failure Patterns

Recognizing these patterns early saves time.

### The kitchen sink session

You start with one task, ask something unrelated, go back to the first task. Context is full of irrelevant information and response quality drops.

**Fix:** `/clear` between unrelated tasks.

### Correcting over and over

Claude does something wrong, you correct it, it's still wrong, you correct again. Context is polluted with failed approaches and Claude keeps trying variations of the same mistake.

**Fix:** after two failed corrections, `/clear` and write a better initial prompt incorporating what you learned.

### The over-specified CLAUDE.md

CLAUDE.md is too long, so important rules get lost. Claude ignores half of it.

**Fix:** ruthlessly prune. If Claude already does something correctly without the instruction, delete it. Convert critical rules to hooks so they're enforced deterministically.

### The trust-then-verify gap

Claude produces a plausible-looking implementation that doesn't handle edge cases — and you ship it without checking.

**Fix:** always provide verification (tests, scripts, screenshots). If you can't verify it, don't ship it.

### The infinite exploration

You ask Claude to "investigate" something without scoping it. Claude reads hundreds of files, filling the context with exploratory work you didn't need.

**Fix:** scope investigations narrowly, or use subagents so the exploration doesn't consume your main context.

---

## Global Reference

| Concept | Key takeaway |
|---------|-------------|
| **Agent** | Claude Code acts directly on your files and commands — no copy-pasting |
| **Agentic loop** | Gather → Act → Verify → repeat if needed |
| **Context window** | The most important resource — everything else follows from this |
| **Plan Mode** | `Shift+Tab` — explores and plans before coding. Essential for complex tasks |
| **Core workflow** | Explore → Plan → Code → Commit |
| **Good prompts** | Precise, with rich context, success criteria, `@` references |
| **Verification** | Give Claude a test to run — evidence, not assertions |
| **CLAUDE.md** | Short, actionable, for things Claude can't guess |
| **Hooks** | Deterministic guardrails — critical rules, not preferences |
| **Subagents** | Isolated context for delegating without polluting the main session |
| **MCP** | Connect to external tools (Linear, Slack, GitHub...) |
| **Skills** | On-demand expertise — loaded only when relevant |
| **Checkpoints** | `Esc+Esc` or `/rewind` — try risky things, rewind if needed |
| **Sessions** | `--continue`, `--resume`, `/rename` — resume context without re-explaining it |
| **Scale** | `claude -p` loops, parallel sessions, Writer/Reviewer pattern |
