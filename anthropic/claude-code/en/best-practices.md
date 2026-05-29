# Claude Code — Best Practices

> Practical patterns for getting the most out of Claude Code, from effective prompting to scaling across parallel sessions.

**Source:** [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) — Official documentation  
**Prerequisites:** [Claude Code 101](./claude-code-101.md) · [Introduction to Agent Skills](./introduction-to-agent-skills.md)

---

## Table of Contents

1. [The One Constraint That Drives Everything](#1-the-one-constraint-that-drives-everything)
2. [Give Claude a Way to Verify Its Work](#2-give-claude-a-way-to-verify-its-work)
3. [Provide Rich, Specific Context](#3-provide-rich-specific-context)
4. [Configure Your Environment](#4-configure-your-environment)
5. [Let Claude Interview You](#5-let-claude-interview-you)
6. [Manage Your Session Actively](#6-manage-your-session-actively)
7. [Automate and Scale](#7-automate-and-scale)
8. [Common Failure Patterns](#8-common-failure-patterns)

---

## 1. The One Constraint That Drives Everything

Almost every best practice in this guide flows from a single constraint: **Claude's context window fills up fast, and performance degrades as it fills.**

Everything Claude "sees" during a session — your messages, every file it reads, every command output — occupies space in its context window. A single debugging session can consume tens of thousands of tokens. As the window fills, Claude may start forgetting earlier instructions or making more mistakes.

**The context window is the most important resource to manage.** The sections below all return to this point.

---

## 2. Give Claude a Way to Verify Its Work

> Give Claude a check it can run: tests, a build, a screenshot to compare. It's the difference between a session you watch and one you walk away from.

Claude stops when the work *looks* done. Without a check it can run, you become the verification loop — every mistake waits for you to notice it.

The check is anything that returns a pass or fail Claude can read: a test suite, a build exit code, a linter, a script that diffs output against a fixture.

### Before / After examples

| Strategy | Weak prompt | Strong prompt |
|----------|-------------|---------------|
| **Verification criterion** | *"implement a validateEmail function"* | *"write validateEmail. test cases: user@example.com → true, invalid → false, user@.com → false. run the tests after implementing"* |
| **Visual UI check** | *"make the dashboard look better"* | *"[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them"* |
| **Root cause** | *"the build is failing"* | *"the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error"* |

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

## 3. Provide Rich, Specific Context

> The more precise your instructions, the fewer corrections you'll need.

Claude can infer intent, but it can't read your mind. Reference specific files, mention constraints, and point to example patterns in your codebase.

### Prompting patterns

| Strategy | Weak | Strong |
|----------|------|--------|
| **Scope the task** | *"add tests for foo.py"* | *"write a test for foo.py covering the edge case where the user is logged out. avoid mocks."* |
| **Point to sources** | *"why does ExecutionFactory have such a weird api?"* | *"look through ExecutionFactory's git history and summarize how its api came to be"* |
| **Reference patterns** | *"add a calendar widget"* | *"look at HotDogWidget.php to understand our widget pattern, then implement a calendar widget that lets the user pick a month and paginate by year. no new libraries."* |
| **Describe the symptom** | *"fix the login bug"* | *"users report login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test, then fix it"* |

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

## 4. Configure Your Environment

### CLAUDE.md: what to include and what to cut

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

**Tip:** if Claude keeps breaking a rule despite it being in `CLAUDE.md`, the file is probably too long and the rule is getting lost. Consider converting critical rules to hooks instead (see [Claude Code 101 — Hooks](./claude-code-101.md#84-hooks)).

### Permission modes

By default, Claude requests approval before every file write or command. After the tenth approval, you're no longer really reviewing — you're just clicking through.

| Mode | How | When to use |
|------|-----|-------------|
| **Approval (default)** | Asks before each action | New tasks, unfamiliar codebases |
| **Auto mode** | Classifier blocks risky actions; routine work proceeds without prompts | Trust the direction, don't want to click every step |
| **Allowlists** | Permit specific safe commands like `npm run lint` or `git commit` | Known safe operations you run constantly |
| **Sandboxing** | OS-level isolation restricts filesystem and network access | Unattended runs with external or untrusted inputs |

Configure via `/permissions` or directly in `.claude/settings.json`.

---

## 5. Let Claude Interview You

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

## 6. Manage Your Session Actively

### Course-correct early

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
- From there: restore conversation only, restore code only, restore both, or summarize from a message
- Checkpoints persist across sessions — close your terminal and rewind later

This means you can tell Claude to try something risky. If it doesn't work, rewind and try a different approach.

> Checkpoints only track changes made *by Claude*, not external processes. This is not a replacement for git.

### Resuming sessions

Claude Code saves conversations locally. You don't have to re-explain context when returning to a task.

```bash
claude --continue        # Pick up the most recent session
claude --resume          # Choose from a list of saved sessions
```

Use `/rename` to give sessions descriptive names like `oauth-migration` so you can find them later.

### Side questions with `/btw`

For quick questions you don't need in context, use `/btw`. The answer appears in a dismissible overlay and never enters conversation history, so you can check a detail without growing context.

```
/btw what does the --no-ff flag do in git merge?
```

### Use subagents for investigation

When Claude researches a codebase, it reads dozens of files — all of which consume context. Subagents explore in a separate context window and report back summaries.

```
Use subagents to investigate how our authentication system handles token
refresh, and whether we have any existing OAuth utilities I should reuse.
```

The subagent reads everything, you get only the conclusion.

---

## 7. Automate and Scale

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

### Writer / Reviewer pattern

A fresh context improves code review — Claude won't be biased toward code it just wrote.

| Session A (Writer) | Session B (Reviewer) |
|--------------------|----------------------|
| `Implement a rate limiter for our API endpoints` | |
| | `Review the rate limiter implementation in @src/middleware/rateLimiter.ts. Look for edge cases, race conditions, and consistency with our existing middleware patterns.` |
| `Here's the review feedback: [output]. Address these issues.` | |

You can do the same with tests: one Claude writes tests, another writes code to pass them.

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

**Step 3 — Test on 2-3 files, then scale.** Refine your prompt based on what goes wrong first, then run on the full set.

### Adversarial review

Before treating a task as done, have a subagent review the result in a fresh context.

```
Use a subagent to review the rate limiter diff against PLAN.md. Check that
every requirement is implemented, the listed edge cases have tests, and
nothing outside the task's scope changed. Report gaps, not style preferences.
```

Because the reviewer runs as a subagent, the implementing session receives the gaps directly and can fix them and re-review without you copying findings between windows.

> A reviewer prompted to find gaps will usually report some, even when the work is sound. Tell it to flag only gaps that affect correctness or stated requirements — not style preferences or hypothetical edge cases.

---

## 8. Common Failure Patterns

Recognizing these patterns early saves time.

### The kitchen sink session

You start with one task, ask something unrelated, go back to the first task. Context is full of irrelevant information and response quality drops.

**Fix:** `/clear` between unrelated tasks.

### Correcting over and over

Claude does something wrong, you correct it, it's still wrong, you correct again. Context is polluted with failed approaches and Claude keeps trying variations of the same mistake.

**Fix:** After two failed corrections, `/clear` and write a better initial prompt incorporating what you learned.

### The over-specified CLAUDE.md

CLAUDE.md is too long, so important rules get lost. Claude ignores half of it.

**Fix:** Ruthlessly prune. If Claude already does something correctly without the instruction, delete it. Convert critical rules to hooks so they're enforced deterministically.

### The trust-then-verify gap

Claude produces a plausible-looking implementation that doesn't handle edge cases — and you ship it without checking.

**Fix:** Always provide verification (tests, scripts, screenshots). If you can't verify it, don't ship it.

### The infinite exploration

You ask Claude to "investigate" something without scoping it. Claude reads hundreds of files, filling the context with exploratory work you didn't need.

**Fix:** Scope investigations narrowly, or use subagents so the exploration doesn't consume your main context.

---

## Quick Reference

| Pattern | Mechanism |
|---------|-----------|
| Verify work automatically | Stop hook, `/goal`, or subagent verifier |
| Avoid context bloat during research | Use subagents |
| Keep critical rules enforced | Hooks, not CLAUDE.md |
| Recover from wrong direction | `Esc+Esc` or `/rewind` |
| Resume across sittings | `claude --continue` or `claude --resume` |
| Quick question without context growth | `/btw` |
| Run Claude in CI | `claude -p "prompt" --output-format json` |
| Parallel workstreams | Worktrees + multiple sessions |
| Unbiased code review | Writer/Reviewer pattern |
| Large-scale migration | `claude -p` loop with `--allowedTools` |
