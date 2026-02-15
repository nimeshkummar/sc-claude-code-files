# Quiz: GitHub Integration and Hooks

## Questions

### Q1: The architectural shift

What's the fundamental architectural shift that the GitHub integration represents? The agent was something before -- what does it become, and what changes about your workflow as a result?

### Q2: Issue-to-PR pipeline

You tag `@Claude` in a GitHub Issue describing a bug fix. Walk through the full pipeline -- what happens from the moment you submit that Issue to the moment you review the result?

### Q3: Lifecycle events

What are the three lifecycle events where you can intercept the agent's control loop, and give a real-world use case for each?

### Q4: Hooks vs CLAUDE.md

What's the architectural difference between telling the agent "always run the linter after editing" in a `CLAUDE.md` instruction vs. configuring it as a hook?

### Q5: Security risk

Hooks execute arbitrary shell commands. What's the security risk, and what specific configuration practice prevents it?

---

## Answers

### A1

The agent moves from a **local pair programmer** (interactive, synchronous, you wait for it) to an **autonomous component of your DevOps pipeline** (headless, asynchronous, triggered by GitHub events).

Workflow shift: from "chat-and-wait" to "assign-and-forget." Write the spec in a GitHub Issue, tag `@Claude`, walk away. The agent delivers a PR without you sitting in a terminal.

Bonus: a second Claude instance auto-reviews the first one's PR -- AI-to-AI validation before human sign-off.

### A2

Full pipeline:

```
Write Issue spec → Tag @Claude → Agent reads issue as prompt → Spins up environment (GitHub Actions runner) → Implements the fix → Opens a PR linked to that issue → Auto-review (second Claude instance) → Human sign-off
```

Key details:
- The agent spins up its own environment -- it runs in a GitHub Actions runner, not your machine
- The PR is linked to the issue -- traceability from spec to implementation
- Auto-review is built in -- a separate Claude instance scans for bugs, security, and code quality as a "Level 1" reviewer

The issue isn't task tracking -- it's a **task execution trigger**.

### A3

Three lifecycle events (not to be confused with matchers/tools):

| Event | When it fires | Use case |
|-------|---------------|----------|
| `PreToolExecution` | Before a tool runs | Block `rm -rf` or prevent access to `.env` files |
| `PostToolUse` | After a tool completes | Run `ruff check` after every file edit |
| `PostPrompt` | After user submits input | Log prompts, inject context |

Important distinction:
- **Lifecycle event** = *when* the hook fires (Pre, Post, PostPrompt)
- **Matcher** = *which tool* triggers it (Read, Edit, Grep)
- **Command** = *what* runs (shell command)

They compose: `PostToolUse` + matcher `Edit` + command `ruff check` = "after every file edit, lint it."

### A4

`CLAUDE.md` is a **suggestion** -- the agent can follow it, forget it, or deprioritize it as context grows. A hook is a **hard guardrail** -- it executes at the control loop level. The agent's own runtime rejects bad output and forces self-correction before you ever see it.

Prompt-level governance vs. programmatic governance.

### A5

The risk is **Remote Code Execution (RCE)**. If you blindly import shared hook configurations from public repositories, you're letting someone else's shell commands run on your machine.

Prevention: keep `.claude/settings.local.json` secure and review any hook configs before using them. Never copy-paste hook configurations from untrusted sources without reading the commands they execute.
