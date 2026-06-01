# Speech: What Is an Agentic CLI and How to Use It Well

> A 3–4 minute talk for a developer team. Trim the "parallel agents" and "automations" paragraphs to stay under 3 minutes.

---

Most of us have used AI by copy-pasting code into a chat window. You get an answer, you paste it in, you test, you go back. That's an *assistant* — a back-and-forth.

An **agentic CLI** is a fundamentally different thing. It has direct access to your dev environment. It reads your files, edits them, runs terminal commands, and loops until the work is done. You give it an objective — it figures out how to get there. No copy-pasting.

---

Before we get to the workflow, one thing to understand first — and this is the most important concept in the whole guide: **the context window**.

Everything the agent sees in a session — your messages, every file it reads, every command output — fills up a finite space. When that space runs low, quality drops. The agent starts making mistakes or forgetting earlier instructions.

Every good practice in this guide exists to protect that space. Keep that in mind as we go through the rest.

---

Now, the core workflow. Four steps: **Explore → Plan → Code → Commit**.

Start with **Explore**: before touching any code, give the agent context. Ask it to read the relevant parts of the codebase and explain what it found. This phase should be read-only.

Then **Plan**: the agent proposes an action plan. This is your best moment to course-correct — before any code is written. Review it carefully, ask questions, push back. A good plan makes the implementation almost mechanical.

Then **Code**: the agent implements the plan. Give it a success criterion — a test to run, a build to pass, a screenshot to compare. Without that, *you* become the verification loop and every mistake waits for you to notice it.

Finally, **Commit**: before pushing, delegate the review to a parallel agent — a fresh instance with no context bias. It reads the diff cold, like a colleague who wasn't in the session. This "Writer/Reviewer" pattern consistently catches things the coding agent missed.

---

A few practices that make a big difference:

**Write precise prompts.** A vague prompt forces the agent to explore broadly to guess your intent. That wastes context. Point to specific files, describe the exact symptom, give a verification criterion at the end.

**Use a project config file** — `CLAUDE.md`, `.cursorrules`, whatever your tool calls it. It's the agent's onboarding doc. Keep it short and actionable: build commands, project conventions, architectural quirks. If removing a line wouldn't cause a mistake, cut it.

**Use deterministic automations** for rules that must apply every time without exception — auto-formatting, blocking edits to prod config, running tests at end of session. A rule in the config file the agent follows *most of the time*. An automation runs *every time*.

**Parallel agents** for anything that involves heavy exploration: codebase research, code review, verification. They run in their own isolated context and return only the result. Your main session stays clean.

---

And the most common failure to avoid: the **correction spiral**. The agent does something wrong, you correct it, it's still wrong, you correct again. After two failed corrections, stop. Reset the context, write a better initial prompt incorporating what you learned. A clean session with a better prompt almost always outperforms a long session patched with corrections.

---

That's the core of it. The guide covers all of this in depth — from permission modes to running agents headlessly in CI — but if you take one thing away: manage your context actively, give the agent a way to verify its work, and use the Explore → Plan → Code → Commit loop. Everything else builds on that.
