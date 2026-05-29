# Agentic CLI — Foundational Guide

> This guide covers the universal concepts that apply to any agentic CLI: Claude Code, OpenCode, Cursor, Copilot CLI, and tools yet to come. Read it before diving into any tool-specific documentation.

---

## Table of Contents

**Part 1 — Understanding an Agentic CLI**
1. [Agent vs Assistant: The Fundamental Difference](#1-agent-vs-assistant-the-fundamental-difference)
2. [The Central Constraint: the Context Window](#2-the-central-constraint-the-context-window)

**Part 2 — Working with an Agent**
3. [The Explore → Plan → Code → Commit Workflow](#3-the-explore--plan--code--commit-workflow)
4. [Writing Good Prompts](#4-writing-good-prompts)
5. [Managing Context Actively](#5-managing-context-actively)

**Part 3 — Verification and Quality**
6. [Give the Agent a Way to Verify Its Work](#6-give-the-agent-a-way-to-verify-its-work)
7. [Code Review with an Agent](#7-code-review-with-an-agent)

**Part 4 — Configuring Your Environment**
8. [The Project Configuration File](#8-the-project-configuration-file)
9. [Permission Modes](#9-permission-modes)
10. [Deterministic Automations](#10-deterministic-automations)
11. [Reusable Instruction Modules](#11-reusable-instruction-modules)
12. [Isolated Parallel Agents](#12-isolated-parallel-agents)
13. [External Tool Connectors](#13-external-tool-connectors)

**Part 5 — Advanced Mastery**
14. [Let the Agent Interview You](#14-let-the-agent-interview-you)
15. [Managing Your Session Actively](#15-managing-your-session-actively)
16. [Automate and Scale](#16-automate-and-scale)
17. [Common Failure Patterns](#17-common-failure-patterns)

[Reference Summary](#reference-summary)

---

## Part 1 — Understanding an Agentic CLI

## 1. Agent vs Assistant: The Fundamental Difference

### What you've probably already used

Most developers have used an AI assistant by copy-pasting code into a chat interface. The assistant responds, you copy, you paste, you test, you go back. It's a manual back-and-forth.

### What an agentic CLI does differently

An agentic CLI has **direct access** to your development environment. It doesn't need you to show it the code — it reads files itself, edits them, and runs commands. You describe an objective; the agent figures out how to achieve it.

Concretely, an agent can:

- **Read and understand your codebase** — explore files, trace a bug, understand an architecture
- **Edit files across your project** — refactor a function and update every file that references it
- **Run terminal commands** — execute tests, install dependencies, read logs, and use the output to decide what to do next
- **Search for information** — look up documentation or API references

### The agentic loop

Every time you send a message, the agent follows this loop:

```
Your prompt
    ↓
Context gathering (reading files, search...)
    ↓
Action (file edit, command execution...)
    ↓
Verification (do the results meet the goal?)
    ↓
Yes → waits for your next message
No  → loops back and tries again
```

You can interrupt or steer the agent at any point in this loop.

### Three things to keep in mind

**The context window is its working memory.** The agent can hold a lot in mind at once, but not your entire project. It explores your codebase strategically to find what it needs.

**It asks for your approval before acting.** By default, most agents wait for your confirmation before editing a file or running a command. You stay in control.

**It can make mistakes.** Like any tool, it can misinterpret a request or introduce a bug. Staying in the loop lets you catch errors early.

---

## 2. The Central Constraint: the Context Window

Almost every best practice in this guide flows from a single constraint: **the agent's context window fills up fast, and performance degrades as it fills.**

Everything the agent "sees" during a session — your messages, every file it reads, every command output — occupies space in its context window. A single debugging session can consume tens of thousands of tokens. As the window fills, the agent may start forgetting earlier instructions or making more mistakes.

**The context window is the most important resource to manage.**

Keep this in mind: every technique in this guide — precise prompts, parallel agents, on-demand instruction modules, context reset commands — aims, directly or indirectly, to preserve that space for what actually matters.

---

## Part 2 — Working with an Agent

## 3. The Explore → Plan → Code → Commit Workflow

This is the core workflow to adopt. It prevents the main trap: asking for code immediately without establishing context, which leads to costly corrections mid-session.

### Explore

Before touching any code, give the agent the context it needs. Most tools offer a read-only mode for this phase. You can also trigger exploration explicitly to get an architecture summary without any intention to make changes.

```
I need to add WebP conversion to our image upload pipeline.
Tell me where in the pipeline it should happen, what dependencies
would be needed, and how you'd approach the implementation.
```

### Plan

The agent reads relevant files, does research if needed, and submits an action plan. This is the **best time to course-correct** — before any code is written. Ask questions, request revisions on specific points. A well-validated plan leads to a clean implementation.

> **Tip:** ask the agent to save the plan to a file (`PLAN.md`, `SPEC.md`…). You'll have a written reference for verification at the end of the task.

### Code

Once the plan is approved, the agent works through the steps. Tips for this phase:

- **Explicit success criterion** — tell the agent what "correct" looks like (green tests, expected behavior, screenshot matching the design)
- **Test suite** — give it as a continuous source of truth; the agent can also write one for you
- **Memorize corrections** — if the agent keeps making the same mistake, add the rule to your project configuration file

### Commit

Before pushing your code:

1. **Run a parallel agent review** — fresh eyes, without the bias of the coding session
2. **Ask the agent to generate a commit message** in your team's style

---

## 4. Writing Good Prompts

The level of precision in your prompt directly determines result quality and how much context gets consumed. A vague prompt forces the agent to explore more broadly to guess your intent.

### Prompting patterns

| Strategy | Weak | Strong |
|----------|------|--------|
| **Scope the task** | *"add tests for foo.py"* | *"write a test for foo.py covering the edge case where the user is logged out. avoid mocks."* |
| **Point to sources** | *"why does ExecutionFactory have such a weird api?"* | *"look through ExecutionFactory's git history and summarize how its api came to be"* |
| **Reference patterns** | *"add a calendar widget"* | *"look at HotDogWidget.php to understand our widget pattern, then implement a calendar widget that lets the user pick a month and paginate by year. no new libraries."* |
| **Describe the symptom** | *"fix the login bug"* | *"users report login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test, then fix it"* |
| **Verification criterion** | *"implement validateEmail"* | *"write validateEmail. test cases: user@example.com → true, invalid → false, user@.com → false. run the tests after."* |

### Ways to provide rich context

Most agentic CLIs let you enrich prompts beyond plain text:

- **Reference files** — point to a specific file rather than describing its content
- **Paste images** — screenshots, mockups, diagrams
- **Give URLs** — documentation links, GitHub issues, API specs
- **Pipe data** — send file content directly (logs, JSON, CSV)
- **Let the agent fetch context** — ask it to use its tools to pull what it needs

### Use existing CLI tools

Agents are effective with CLI tools like `gh` (GitHub), `aws`, `gcloud`. These are more context-efficient than direct integration equivalents, and most agents already know how to use them.

For tools the agent doesn't know: *"Use `foo-cli --help` to learn the tool, then do X with it."*

---

## 5. Managing Context Actively

### The fundamental operations

Every agentic tool offers equivalents of these three operations:

| Operation | Effect | When to use |
|-----------|--------|-------------|
| **Compact** | Summarizes and compresses history to free up space | Mid-feature, when approaching the limit |
| **Clear** | Wipes everything, fresh start | Between distinct features |
| **Inspect** | Shows current context usage | To diagnose what's consuming space |

> **Rule of thumb:** compact during a feature, clear between features. What you want the agent to remember across sessions → put it in your project configuration file.

### Strategies to save context

**Be specific in your prompts.** A vague prompt forces the agent to explore more broadly to guess your intent.

**Disable unused integrations.** Every activated connector loads its definitions into context, even when you're not using it.

**Delegate research to a parallel agent.** When you need to explore your codebase for a specific answer, a parallel agent does the work in its own context and returns only the result.

---

## Part 3 — Verification and Quality

## 6. Give the Agent a Way to Verify Its Work

> Give the agent a check it can run: tests, a build, a screenshot to compare. It's the difference between a session you watch and one you can walk away from.

The agent stops when the work *looks* done. Without an automated check, you become the verification loop — every mistake waits for you to notice it.

### Levels of verification

| Level | Mechanism | Setup | Best for |
|-------|-----------|-------|----------|
| **In the prompt** | Ask the agent to run the check and iterate | None | Any task |
| **Session condition** | Goal defined at session start — agent re-checks after each action | Low | Long tasks |
| **Deterministic gate** | Automation that blocks the turn from ending until the check passes | Medium | Unattended runs |
| **Second opinion** | Parallel verification agent with a fresh context | Medium | Critical code |

### Ask for evidence, not assertions

Have the agent show you the test output, the command it ran, or a screenshot — rather than just saying "done". Reviewing evidence is faster than re-running the verification yourself.

---

## 7. Code Review with an Agent

### The fresh context principle

Before pushing a PR, delegate the review to a parallel agent. The key advantage: the parallel agent starts with a clean context, without the bias built up during the coding session.

**Recommended configuration:**
- Read-only tools only — a reviewer flags issues, it doesn't fix them
- Version the configuration in your repo so the whole team uses the same reviewer

### Writer / Reviewer pattern

A fresh context improves code review — the agent won't be biased toward code it just wrote.

| Session A (Writer) | Session B (Reviewer) |
|--------------------|----------------------|
| *Implement the rate limiter* | |
| | *Review the implementation in [file]. Look for edge cases, race conditions, consistency with existing middleware.* |
| *Here's the feedback: [output]. Address these issues.* | |

You can do the same with tests: one agent writes tests, another writes code to pass them.

### Adversarial review

Before treating a task as done, have a parallel agent review the result in a fresh context, giving it the original plan as reference.

```
Review the diff against PLAN.md. Check that every requirement is implemented,
the listed edge cases have tests, and nothing outside scope changed.
Report gaps, not style preferences.
```

> A reviewer prompted to find gaps will usually report some, even when the work is sound. Tell it to flag only gaps that affect correctness or stated requirements.

---

## Part 4 — Configuring Your Environment

## 8. The Project Configuration File

Every major agentic CLI offers a configuration file that the agent reads automatically at the start of every session. It's the **onboarding document for the agent** — it prevents rediscovering the same things every time.

### Equivalents by tool

| Tool | Configuration file |
|------|-------------------|
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursorrules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| OpenCode | `AGENTS.md` |

### What to include

| ✅ Include | ❌ Exclude |
|-----------|----------|
| Build, test, lint commands | Anything the agent can infer from reading code |
| Code style rules that differ from defaults | Standard language conventions |
| Branch naming and PR conventions | Detailed API documentation (link instead) |
| Architectural decisions specific to your project | Information that changes frequently |
| Developer environment quirks (required env vars) | Long explanations or tutorials |

**Rule of thumb:** for each line, ask *"Would removing this cause the agent to make mistakes?"* If not, cut it. A bloated file causes the agent to ignore rules buried in the noise.

### Practical tips

**Let the content emerge.** Start without a configuration file and observe where you keep correcting the agent. Those corrections are worth memorializing.

**Multiple scopes.** Most tools allow a global file (all your projects) and a project file (whole team via git). Use them complementarily.

**Memorize recurring corrections.** If you find yourself repeating the same instruction, add the rule to the configuration file.

---

## 9. Permission Modes

By default, the agent requests approval before every file write or command. After the tenth approval, you're no longer really reviewing — you're just clicking through.

### Control spectrum

```
Maximum control ←————————————————→ Maximum autonomy
    Approval      Selective        Auto    Sandbox
    (default)     allowlists       mode
```

| Mode | Behavior | When to use |
|------|----------|-------------|
| **Approval** | Asks before each action | New tasks, unfamiliar codebases |
| **Allowlists** | Specific known-safe commands run automatically | Repetitive, well-understood operations |
| **Auto / unsupervised** | Tool manages approvals with a risk classifier | Long tasks in a trusted direction |
| **Sandbox** | OS-level isolation, restricted filesystem and network | Runs with external or unverified inputs |

---

## 10. Deterministic Automations

Most agentic CLIs let you define scripts that run automatically at specific points in the workflow. Their strength: **they are deterministic**. Putting a rule in the configuration file → the agent follows it *most of the time*. An automation → the rule applies **every time, without exception**.

### Common trigger points

| Event | Example uses |
|-------|-------------|
| Before an action | Block modifications to sensitive files |
| After an action | Auto-format edited code |
| On prompt submission | Enrich or validate the prompt |
| At end of response | Run tests, send a notification |

### Typical team use cases

- **Auto-formatting**: after each file edit → Prettier, `gofmt`, etc.
- **Audit logging**: record every command the agent executes
- **Guardrails**: block edits to prod config, `rm -rf`, direct commits to `main`
- **Verification**: run tests at end of session and block if they fail
- **Notifications**: Slack or desktop alert when a long task completes

> **Principle:** what must happen every time without exception → a deterministic automation. What is a preference → the configuration file.

---

## 11. Reusable Instruction Modules

Most agentic CLIs let you define sets of specialized instructions that are only loaded on demand, when relevant.

### The problem they solve

Your PR review checklist doesn't need to be in memory when you're debugging a performance issue. Reusable instruction modules are loaded **only when the context justifies it**, preserving context window space.

### Equivalents by tool

| Tool | Mechanism |
|------|-----------|
| Claude Code | Skills (`SKILL.md`) |
| Cursor | Rules by context |
| OpenCode | Custom instructions |

### Design best practices

- **The description is crucial** — it's what the agent reads to decide if the module is relevant. A vague description = a module that never triggers.
- **Keep them focused** — one module = one domain of expertise. Avoid catch-all modules.
- **Share via git** — versioning modules in the repo means the whole team benefits automatically.

---

## 12. Isolated Parallel Agents

A parallel agent is an agent instance that runs with its own isolated context. It receives a task, processes it, and returns only its result — without polluting your main context.

### When to use them

| Use case | Why a parallel agent |
|----------|---------------------|
| Codebase exploration | Reads dozens of files without consuming your context |
| Code review | Fresh context = no bias toward code it just wrote |
| Verification | A different agent from the one who implemented evaluates the result |
| Independent tasks | Multiple workstreams in parallel |

### Equivalents by tool

| Tool | Mechanism |
|------|-----------|
| Claude Code | Subagents (`.claude/agents/`) |
| OpenCode | Specialized agents |
| Cursor | Composer agents |

---

## 13. External Tool Connectors

Most modern agentic CLIs support a standard for connecting to external tools and data sources: Linear, Slack, GitHub, databases, documentation...

### MCP — Model Context Protocol

MCP is the emerging open standard, adopted by many tools. It lets the agent interact with external services without you having to copy-paste data.

```bash
# Generic example of adding a connector
<tool> mcp add <server-name>
```

### Scope and sharing

Most tools let you define connectors at multiple levels:

| Scope | Sharing |
|-------|---------|
| Personal | You only |
| Global user | All your projects |
| Project (versioned file) | Whole team automatically |

> To share connectors with the team: commit the MCP configuration file at the repo root.

### Watch your context

Every activated connector loads its tool definitions into context, even when unused. If a tool has a CLI equivalent (`gh`, `aws`...), the CLI is often more context-efficient.

---

## Part 5 — Advanced Mastery

## 14. Let the Agent Interview You

> For larger features, have the agent interview you first — it asks about things you might not have considered yet.

Instead of writing a full spec yourself, start with a minimal description and let the agent extract requirements through structured questions.

```
I want to build [brief description]. Interview me in detail.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
Don't ask obvious questions — dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Once the spec is complete, **start a fresh session** to implement it. The new session has clean context focused entirely on implementation.

**What makes a good spec:** it names the files and interfaces involved, states what is explicitly out of scope, and ends with an end-to-end verification step that proves the feature works.

---

## 15. Managing Your Session Actively

### Course-correct early and often

The best results come from tight feedback loops. Correct the agent as soon as you notice it going off track — waiting is never faster.

| Action | When |
|--------|------|
| **Stop** the agent mid-action | It's doing the wrong thing |
| **Undo** changes and redirect | Wrong approach, want to try differently |
| **Reset** context | New unrelated task, or after two failed corrections |

> **Rule:** if you've corrected the agent more than twice on the same issue, reset and write a better initial prompt incorporating what you learned. A clean session with a better prompt almost always outperforms a long session with accumulated corrections.

### Checkpoints and recovery

Most agentic CLIs automatically create save points before each modification. This lets you:

- **Roll back** file changes without touching the conversation
- **Resume** an interrupted session without re-explaining context
- **Experiment** safely — try something risky, undo if it doesn't work

> Checkpoints only track changes made *by the agent*, not external processes. This is not a replacement for git.

### Resuming sessions

Most tools save conversations locally. You don't have to re-explain context when returning to a task. Give descriptive names to your sessions (`oauth-migration`, `auth-refactor`…) to find them easily later.

---

## 16. Automate and Scale

### Non-interactive mode

Most agentic CLIs offer a headless mode for integration into CI/CD pipelines, pre-commit hooks, or scripts.

```bash
# General principle (syntax varies by tool)
<tool> --non-interactive "your prompt" --output-format json
```

Use this mode to:
- Automatically analyze code on every PR
- Run batch migrations
- Integrate the agent into your existing pipelines

### Parallel sessions

To accelerate development, several approaches:

| Approach | Best for |
|----------|----------|
| **Git worktrees** | Parallel features in isolated checkouts — edits don't collide |
| **Multiple sessions** | Independent workstreams managed visually |
| **Agent teams** | Automated coordination of multiple agents with shared tasks |

### Fan-out at scale

For large migrations or analyses, distribute work across parallel invocations:

1. **Generate a list of files** to process
2. **Write a loop** that invokes the agent in non-interactive mode for each file
3. **Test on 2-3 files**, refine the prompt, then run at scale

```bash
# Principle (adapt syntax to your tool)
for file in $(cat files.txt); do
  <tool> "Migrate $file from X to Y. Return OK or FAIL."
done
```

---

## 17. Common Failure Patterns

Recognizing these patterns early saves time. They are universal — they apply regardless of which tool you use.

### The kitchen sink session

You start with one task, ask something unrelated, go back to the first task. Context is full of irrelevant information and response quality drops.

**Fix:** reset context between unrelated tasks.

### Correcting over and over

The agent does something wrong, you correct it, it's still wrong, you correct again. Context is polluted with failed approaches and the agent keeps trying variations of the same mistake.

**Fix:** after two failed corrections, reset and write a better initial prompt incorporating what you learned.

### The over-specified configuration file

The configuration file is too long, so important rules get lost. The agent ignores half of it.

**Fix:** ruthlessly prune. If the agent already does something correctly without the instruction, delete it. Convert critical rules to deterministic automations.

### The trust-then-verify gap

The agent produces a plausible-looking implementation that doesn't handle edge cases — and you ship it without checking.

**Fix:** always provide verification (tests, scripts, screenshots). If you can't verify it, don't ship it.

### The infinite exploration

You ask the agent to "investigate" something without scoping it. The agent reads hundreds of files, filling the context with exploratory work you didn't need.

**Fix:** scope investigations narrowly, or delegate them to a parallel agent so the exploration doesn't consume your main context.

### The ignored plan

You approve a plan, then let the agent work without intermediate checks. By the end of the session, the implementation has drifted from the plan without you noticing.

**Fix:** give the agent the plan as a reference at each major step. Ask it to verify conformance before moving on.

---

## Reference Summary

| Concept | Key takeaway |
|---------|-------------|
| **Agent vs assistant** | The agent acts directly on your environment — no copy-pasting |
| **Agentic loop** | Gather → Act → Verify → repeat if needed |
| **Context window** | The central resource — everything else follows from this |
| **Core workflow** | Explore → Plan → Code → Commit |
| **Good prompts** | Precise, with rich context, success criteria, file references |
| **Verification** | Give the agent a test to run — evidence, not assertions |
| **Config file** | Short, actionable, for things the agent can't guess |
| **Automations** | For critical rules — deterministic, no exceptions |
| **Parallel agents** | Isolated context for delegating without polluting the main session |
| **Connectors** | Connect to external tools — watch context loaded |
| **Reusable modules** | On-demand expertise — loaded only when relevant |
| **Checkpoints** | Experiment safely — undo if needed |
| **Sessions** | Name them, resume them, don't re-explain them |
| **Scale** | Headless loops, parallel sessions, Writer/Reviewer pattern |

---

*To go further, see the tool-specific documentation:*
- *[Claude Code](../../anthropic/claude-code/en/learning-path.md)*
- *[OpenCode](../../opencode/)*
