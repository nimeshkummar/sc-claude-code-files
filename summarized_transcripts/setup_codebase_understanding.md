# Setup and Codebase Understanding

This transcript outlines the operational framework for Claude Code, focusing on its capacity for autonomous system discovery and architectural governance.

## Systemic Discovery and Mental Models

Claude Code functions as an active agent rather than a passive retrieval system. Instead of relying on a pre-computed vector index, it uses a real-time "agentic search" to traverse the codebase. It identifies entry points, analyzes dependencies, and constructs a mental model of the architecture on the fly.

Key capabilities in this phase include:

- **Traceability:** The agent can trace the lifecycle of a request from the frontend through the API layer to the backend and data persistence layers.
- **Visual Synthesis:** It can generate ASCII art or structured diagrams to visualize complex data flows, allowing you to verify if the implementation aligns with your intended design.
- **Interruption Protocol:** Using the Esc key, you can interrupt the agent's "to-do list" to redirect its investigation, providing the human-in-the-loop oversight necessary for complex systems.

## The Governance Layer: CLAUDE.md

The core of your architectural control resides in the memory hierarchy. The `/init` command bootstraps this by generating a `CLAUDE.md` file. This file is not merely documentation; it is the System Prompt for that specific repository.

There are three distinct layers of instruction:

- **Project Level (`CLAUDE.md`):** Committed to Git. Defines architectural constraints (e.g., "Always use uv for dependency management," "Enforce Hexagonal Architecture patterns").
- **Local Level (`CLAUDE.local.md`):** Gitignored. Specific to your M1/M4 hardware setup or personal workflow.
- **Global Level (`~/.claude/CLAUDE.md`):** Cross-project rules. This is where you should define your preference for POSIX-compliant Markdown and strict architectural decision records.

## Context and Command Control

To maximize the working memory capacity of the agent, the transcript highlights several primitives:

- **`/ide`:** Syncs the agent with VS Code, tagging the current file and line number as immediate context.
- **`#` Shortcut:** Allows for immediate injection of "Memory" into the governance files without leaving the CLI.
- **`/compact`:** Truncates history while retaining a summary of the state. This is essential for managing long-running architectural refactors without saturating the context window.
- **Git Integration:** The agent handles staging and commit message generation, ensuring the history reflects the actual logic changes made during the session.

## Practical Takeaway

Claude Code serves as an implementation agent. By defining the system architecture in `CLAUDE.md`, you can delegate the routine coding tasks while you focus on system design and architectural decisions.

**Next Action:** Run `claude /init` in a repo. Use the `#` shortcut to add your architectural principles to the global `CLAUDE.md` so every project the agent touches is governed by consistent standards.
