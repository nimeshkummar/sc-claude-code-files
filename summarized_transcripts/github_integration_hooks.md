# GitHub Integration and Hooks

This breakdown analyzes the CI/CD Integration and Event-Driven Architecture capabilities of Claude Code. It moves the agent from a local "pair programmer" to an autonomous component of your DevOps pipeline.

## High-Level Architectural Concepts

1. **Headless Agent Integration (CI/CD):** Moving the agent from an interactive CLI session to an asynchronous background worker triggered by GitHub Events (Pull Requests, Issues). This treats "coding" as a microservice running within the GitHub Actions runner.
2. **Asynchronous Delegation:** The shift from synchronous "chat-and-wait" loops to "assign-and-forget" workflows. You define the spec in a GitHub Issue, and the agent asynchronously delivers a PR.
3. **Lifecycle Hooks (Event-Driven Governance):** The ability to inject arbitrary shell execution at specific state changes in the agent's operation (e.g., `PostToolUse`, `PreToolExecution`). This is the governance layer where you enforce architectural constraints programmatically.

---

## 1. State Cleanup and Git Hygiene

- **Context:** The session resumes immediately following the previous lesson's use of Git Worktrees.
- **The Resume Flag:** Using `--resume` allows the agent to reload the context of the previous session, including knowledge of the `.trees` directory structure.
- **Cleanup:** The agent executes `git worktree remove` and directory deletion commands to clean up ephemeral environments before pushing the final merged state to `origin main`.

## 2. The GitHub Integration (Agent as a Service)

The video demonstrates installing the Claude App to enable server-side agentic behaviors.

- **Command:** `/install-GitHub-app`
- **Architecture:** This installs a GitHub App that connects your repository to Anthropic's managed infrastructure. It generates GitHub Actions workflow YAML files inside `.github/workflows/`.

### Capabilities

- **Auto-Review:** The agent automatically scans new PRs for bugs, security issues, and code quality, acting as a "Level 1" reviewer.
- **Issue Actuation:** Tag `@Claude` in a GitHub Issue. The agent reads the issue description as a prompt, spins up an environment, implements the fix, and opens a PR linked to that issue.
- **Self-Correction:** Claude creates a PR, then *another* instance of Claude reviews that PR -- an AI-to-AI validation loop before human sign-off.

### The async workflow

```
Write Issue spec → Tag @Claude → Agent implements → PR opened → Auto-reviewed → Human sign-off
```

This converts GitHub Issues from "task tracking" into "task execution." The issue isn't a reminder to do work -- it *is* the work trigger.

## 3. Operational Hooks (The Control Plane)

The most critical architectural feature: **Hooks** (`/hooks`) allow you to intercept the agent's control loop.

- **Configuration:** Hooks are defined in `.claude/settings.local.json` (local) or project-specific configs.

### Lifecycle events

| Event | When it fires | Example use case |
|-------|---------------|------------------|
| `PreToolExecution` | Before a tool runs | Block `rm -rf` or access to `.env` files |
| `PostToolUse` | After a tool completes | Run linter after every file edit |
| `PostPrompt` | After user submits input | Log prompts, inject context |

### The Matcher pattern

Hooks use matching logic to filter which events trigger execution:

- **Matcher:** Specifies which tools to intercept (e.g., `Read`, `Grep`, `Edit`)
- **Execution:** When the matcher triggers, an arbitrary shell command runs

**Trivial example:** Run `say "All done"` when the agent finishes reading a file (audio notification).

**Real-world example:** Configure a `PostToolUse` hook on the `Edit` tool to run `ruff check` or `terraform fmt`. If the linter fails, the hook returns an error, forcing the agent to self-correct immediately -- bad code never persists in the context.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "tool": "Edit" },
        "command": "ruff check --fix $FILE"
      }
    ]
  }
}
```

### Why hooks matter architecturally

Hooks turn "please follow the style guide" (a suggestion the agent can ignore) into a **hard guardrail** (a programmatic check the agent cannot bypass). The difference:

- Without hooks: you review the output and ask the agent to fix linting errors after the fact
- With hooks: the agent's own control loop rejects bad output and self-corrects before you ever see it

## Security Warning

Hooks execute arbitrary shell commands. This is a vector for Remote Code Execution (RCE) if you blindly import shared hook configurations from public repositories. Keep `.claude/settings.local.json` secure and review any hook configs before using them.

## Key Takeaways

- **GitHub integration turns Issues into execution triggers.** Write the spec in an Issue, tag `@Claude`, walk away. The agent delivers a PR asynchronously.
- **Auto-review creates an AI-to-AI validation loop.** The implementing agent and the reviewing agent are separate instances -- one builds, the other critiques, before human sign-off.
- **Hooks are programmatic governance.** They enforce standards at the control loop level, not the prompt level. Linters, formatters, and security scanners run automatically on every edit.
- **`--resume` preserves session state.** Critical for multi-step workflows that span multiple sessions (e.g., worktree cleanup after parallel work).
- **Treat hook configs as security-sensitive.** Arbitrary shell execution + untrusted configs = RCE.
