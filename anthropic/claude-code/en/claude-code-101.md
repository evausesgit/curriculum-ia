# Claude Code 101 — Team Learning Guide

> This guide teaches development teams how to use Claude Code effectively in their daily workflow. It covers core concepts, practical workflows, and customization tools.

**Original course:** [Claude Code 101](https://anthropic.skilljar.com/claude-code-101) — Anthropic Academy (free)  
**Format:** 13 lessons · 1.5h of video · quiz · certificate

---

## Table of Contents

1. [What Claude Code Actually Is](#1-what-claude-code-actually-is)
2. [How It Works Under the Hood](#2-how-it-works-under-the-hood)
3. [Installation by Environment](#3-installation-by-environment)
4. [Writing Your First Prompt](#4-writing-your-first-prompt)
5. [The Explore → Plan → Code → Commit Workflow](#5-the-explore--plan--code--commit-workflow)
6. [Managing Context in Long Sessions](#6-managing-context-in-long-sessions)
7. [Code Review with Claude](#7-code-review-with-claude)
8. [Customizing Claude Code for Your Team](#8-customizing-claude-code-for-your-team)

---

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

### Three things to keep in mind

**The context window is its working memory.** Claude can hold a lot in mind at once, but not your entire project. It explores your codebase strategically to find what it needs.

**It asks for your approval before acting.** By default, Claude Code waits for your confirmation before editing a file or running a command. You stay in control.

**It can make mistakes.** Like any tool, it can misinterpret a request or introduce a bug. Staying in the loop lets you catch errors early.

---

## 2. How It Works Under the Hood

Understanding Claude Code's internals makes it significantly easier to use well.

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

### The context window

Everything Claude "sees" during a session — your messages, files read, command outputs — occupies space in its context window. When it fills up, Claude automatically compacts past exchanges to free up space. Some details can be lost in the process, which is why managing context matters (see section 6).

### Tools

Tools are what turn Claude from a text generator into an agent that can act. Each capability — reading a file, running a command, searching the web — is a tool that Claude selects based on the situation.

### Permission modes

Claude Code offers several levels of autonomy:

| Mode | Behavior |
|------|----------|
| **Approval (default)** | Asks for confirmation before each file edit or command |
| **Auto-accept** | Edits files without asking, but still asks for commands |
| **Plan Mode** | Uses only read-only tools to build a plan before any action |

> **Team tip:** start in Approval mode while getting familiar with what Claude does, then switch to Auto-accept for routine tasks.

---

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

Open Claude Desktop → enable the **"Code"** toggle at the top. Lets you work in a specific folder with configurable permissions, and run Claude in the background while you work on other things.

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

## 4. Writing Your First Prompt

### Choosing your interaction mode

`Shift + Tab` cycles through available modes:

- **Approval** → Claude asks before each action
- **Auto-accept** → file edits happen automatically
- **Plan Mode** → Claude analyzes and plans before touching anything

### Plan Mode: essential for complex tasks

Plan Mode is the right choice for multi-step implementations. In this mode, Claude uses only read-only tools to explore your code and ask clarifying questions, then submits a detailed plan for your approval before writing a single line.

**Example prompt in Plan Mode:**
```
My app needs a dark mode system.
Create a toggle in the header that switches between light and dark mode
across the entire app, consistent with my existing color palette.
```

Claude will explore your CSS/Tailwind files, ask questions if needed, then walk you through a step-by-step plan before any code is written.

### Prompt best practices

- **Be specific** — a vague prompt forces Claude to explore more broadly, consuming more context
- **Define a success criterion** — "tests must pass" or "the render must match the design"
- **Mention constraints** — framework, version, project conventions

---

## 5. The Explore → Plan → Code → Commit Workflow

This is the core workflow for teams to adopt. It prevents the main trap: asking for code immediately without establishing context, which leads to costly corrections mid-session.

### Explore

Before touching any code, give Claude the context it needs. In Plan Mode, Claude explores your codebase read-only. You can also trigger exploration explicitly just to get an architecture summary, without any intention to make changes.

**Example:**
```
I need to add WebP conversion to our image upload pipeline.
Tell me where in the pipeline it should happen, what dependencies
would be needed, and how you'd approach the implementation.
```

### Plan

Claude reads relevant files, does research if needed, and submits an action plan. This is the **best time to course-correct** — before any code is written. Ask questions, request revisions on specific points. A well-validated plan leads to a clean implementation.

### Code

Once the plan is approved, Claude works through the steps. Tips for this phase:

- **Explicit success criterion** — tell Claude what "correct" looks like (green tests, expected behavior...)
- **Test suite** — give Claude a test suite to continuously validate against. It can also write one for you
- **Right tools** — for web UIs, the Claude in Chrome extension lets Claude control a browser tab and test visually
- **Memorize corrections** — if Claude keeps making the same mistake, ask it to save the fix to `CLAUDE.md`

### Commit

Before pushing your code:

1. **Run a subagent review** (see section 7) — fresh eyes, without the bias of the coding session
2. **Ask Claude to generate a commit message** in your team's style

---

## 6. Managing Context in Long Sessions

The context window is a finite resource. Managing it actively maintains response quality throughout a session.

### The three essential commands

| Command | Effect | When to use |
|---------|--------|-------------|
| `/compact` | Compacts history into a summary | Mid-feature, when approaching the limit |
| `/clear` | Wipes everything, fresh start | Between distinct features |
| `/context` | Shows size, categories, breakdown | To diagnose what's consuming context |

> **Rule of thumb:** `/compact` during a feature, `/clear` between features. What you want Claude to remember across sessions → put it in `CLAUDE.md`.

### Strategies to save context

**Be specific in your prompts.** A vague prompt forces Claude to explore more broadly to guess your intent — this consumes far more context than a detailed prompt would.

**Disable unused MCP servers.** Every connected MCP server loads its tool definitions into context, even when you're not using it. Check with `/mcp` and disable what isn't relevant to the current task.

**Use subagents for research tasks.** When you need to explore your codebase for a specific answer, a subagent does the work in its own context and returns only the result — without cluttering your main context.

---

## 7. Code Review with Claude

### Subagent review: guaranteed fresh perspective

Before pushing a PR, delegate the review to a subagent. The key advantage: the subagent starts with a clean context, without the bias built up during the coding session.

**Recommended configuration for a review subagent:**
- Read-only tools only — a reviewer flags issues, it doesn't fix them
- Version the configuration in your repo so the whole team uses the same reviewer

### Commit, push, and PR in one command

The `/commit-push-pr` skill chains commit, push, and PR creation in a single step. If you have a Slack MCP server configured with channels listed in `CLAUDE.md`, it can also automatically post the PR link to your team channel.

### Resuming work on an existing PR

When Claude creates a PR via `gh pr create`, the session is linked to that PR. To resume work later:

```bash
claude --from-pr <PR_NUMBER>
```

Useful for addressing review comments or fixing a broken build.

---

## 8. Customizing Claude Code for Your Team

### 8.1 The CLAUDE.md file

`CLAUDE.md` is a Markdown file at the root of your project. Claude reads it automatically at the start of every session. Think of it as an **onboarding document for Claude** — it prevents rediscovering the same things every session.

#### Recommended minimal structure

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

#### Memory file hierarchy

| File | Location | Scope |
|------|----------|-------|
| Project `CLAUDE.md` | Repo root | Whole team (commit to git) |
| Personal `CLAUDE.md` | User config folder | You only, all your projects |

#### Practical tips

**Let the content emerge.** Start without a `CLAUDE.md` and observe where you keep correcting Claude. Those corrections are worth memorializing. Use `/init` to have Claude generate a first version.

**Reference existing documentation:**
```markdown
## Architecture
For more context: @docs/architecture.md
```

**Memorize recurring corrections.** If you find yourself repeating the same instruction, ask Claude: *"Save this rule to the project CLAUDE.md."*

---

### 8.2 Subagents

A subagent is a Claude instance that runs in parallel with its own isolated context. It receives a task, processes it, and returns only its result — without polluting your main context.

#### Why it matters

When you ask Claude to explore your codebase to locate something, Claude reads dozens of files — all of that consumes context. If you only need the final answer, all that exploratory work is noise. A subagent does the same exploration in its own bubble and returns just the conclusion.

#### Creating a subagent

```
/agents → "Create new agent"
```

You define: scope (personal or project), purpose, available tools, and when Claude should call it automatically.

Subagents are Markdown files with YAML frontmatter — versionable in your repo.

---

### 8.3 MCP (Model Context Protocol)

MCP is an open standard that connects Claude Code to external tools and data sources — Linear, Slack, GitHub, dependency documentation, databases...

#### Adding an MCP server

```bash
claude mcp add <server-name>
```

Two types: **HTTP** (remote services) and **Stdio** (local processes).

#### Server scoping

| Scope | Config file | Sharing |
|-------|-------------|---------|
| Local | Personal settings | You only |
| User | Global settings | All your projects |
| Project | `.mcp.json` in repo | Whole team automatically |

> To share MCPs with the team: commit `.mcp.json` at the repo root. Everyone gets the same servers on the next `git pull`.

#### Watch your context

Every connected MCP server loads its tool definitions into context, even when unused. If a tool has a CLI equivalent (`gh`, `aws`...), the CLI is more context-efficient than the corresponding MCP server.

---

### 8.4 Hooks

Hooks are shell commands that run automatically at specific points in Claude Code's lifecycle. Their strength: they are **deterministic**. Putting a rule in `CLAUDE.md` means Claude follows it *most of the time*. A hook means the rule applies **every single time, no exceptions**.

#### Available events

| Event | Trigger |
|-------|---------|
| `PreToolUse` | Before a tool call executes |
| `PostToolUse` | After a tool call completes |
| `UserPromptSubmit` | When you submit a prompt, before Claude processes it |
| `Stop` | When Claude finishes responding |
| `Notification` | When Claude sends a notification |

#### Typical team use cases

- **Auto-formatting**: `PostToolUse` on `Edit|MultiEdit|Write` → run Prettier, `gofmt`, etc.
- **Audit logging**: record every command Claude executes
- **Guardrails**: block edits to prod config, `rm -rf`, direct commits to `main`
- **Notifications**: Slack or desktop alert when a long task completes

#### Blocking with PreToolUse

A `PreToolUse` hook can block an action before it executes. It receives the tool name and its parameters as JSON on `stdin`:

| Exit code | Behavior |
|-----------|----------|
| `0` | Action proceeds normally |
| `2` | Action blocked — `stderr` message is fed back to Claude so it understands why |
| Other | Non-blocking error visible to you only |

#### Sharing hooks with the team

Commit `.claude/settings.json` to your repo. The whole team automatically gets the same guardrails. Use `CLAUDE_PROJECT_DIR` to reference scripts stored in your project — the path works regardless of Claude's current working directory.

> **Principle:** what must happen every time without exception → a hook. What is a preference → `CLAUDE.md`.

---

## Summary

| Concept | Key takeaway |
|---------|-------------|
| **Agent** | Claude Code acts directly on your files and commands — no copy-pasting |
| **Agentic loop** | Gather → Act → Verify → repeat if needed |
| **Context window** | Limited working memory — manage it with `/compact`, `/clear`, `/context` |
| **Plan Mode** | `Shift+Tab` — explores and plans before coding. Essential for complex tasks |
| **Core workflow** | Explore → Plan → Code → Commit |
| **CLAUDE.md** | Persistent project memory — onboarding document for Claude |
| **Subagents** | Isolated context for delegating tasks without polluting the main context |
| **MCP** | Connection to external tools (Linear, Slack, GitHub...) |
| **Hooks** | Deterministic guardrails — run every time, no exceptions |
