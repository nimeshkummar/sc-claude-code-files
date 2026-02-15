# Adding Features with Claude Code

This breakdown analyzes the operational and architectural primitives of Claude Code, focusing on the transition from simple code generation to agentic system manipulation.

## Context Injection Primitives

An agent's performance is a function of its active context. The transcript highlights two methods for managing the agent's working memory:

- **Explicit File Referencing (`@` tagging):** Instead of relying on the agent to search and potentially hallucinate the correct entry points, the `@` symbol allows for surgical context injection. By tagging specific files or folders, you constrain the agent's attention to relevant modules, reducing token noise and increasing implementation accuracy.

- **Context Resetting (`/clear`):** Large context windows eventually suffer from "middle-of-the-prompt" degradation or conflicting previous instructions. Clearing the history before starting a new feature ensures the agent is not polluted by logic used for previous, unrelated features.

## The Planning Protocol (Plan Mode)

The transcript introduces a non-linear development workflow called Plan Mode (`Shift+Tab` twice). This is a critical architectural safeguard for multi-file systems.

**Logic:** Separates "Reasoning" from "Actuation."

**Workflow:**

1. **Ingestion:** The agent reads the provided files.
2. **Synthesis:** It generates a step-by-step roadmap across the frontend and backend.
3. **Human-in-the-loop (HITL):** You review the architectural decisions before any code is written.
4. **Execution:** Upon approval, the agent executes the changes sequentially.

**Tradeoff:** Slower than direct execution but prevents the "cascading error" effect where a mistake in file A breaks the logic in file B.

## Multimodal UI Iteration

The agent utilizes multimodal vision to bridge the gap between code and visual reality.

- **Screenshot Feedback Loop:** When UI elements are visually unappealing (e.g., default blue links on a specific background), you provide a screenshot. The agent analyzes the rendered pixels, maps them back to the underlying CSS/HTML in the source code, and proposes styling changes. This bypasses the need to identify specific CSS classes manually.

## Model Context Protocol (MCP) and Actuation

The most advanced concept presented is using MCP to extend the agent's capabilities beyond the file system.

- **Playwright MCP:** Allows the agent to control a headless browser.
  1. **Navigation:** The agent visits the live application URL.
  2. **Verification:** It takes its own screenshots to verify if a UI element actually appeared and matches the design.
  3. **Autonomous Testing:** It can evaluate JavaScript or click elements to ensure the frontend-backend handshake is functional.

- **Architectural Value:** This converts the agent from a "Code Writer" to a "Systems Tester." It can close the loop by verifying its own implementation in a live environment.

## Architectural Governance (Memory Hierarchy)

The transcript emphasizes `.md` files as a mechanism to enforce system-wide constraints.

| Layer | File | Scope | Example |
|-------|------|-------|---------|
| Project | `CLAUDE.md` | Shared via Git | "Register all new tools in `search_tools.py`" |
| Local | `CLAUDE.local.md` | Git-ignored | "I will start the server myself; do not use `./run.sh`" |
| Global | `~/.claude/CLAUDE.md` | Machine-wide | "Always use POSIX-compliant Markdown" |

## Backend Tool Expansion

The final section demonstrates agentic expansion of a RAG (Retrieval-Augmented Generation) system.

1. **Tool Registration:** The agent modifies `search_tools.py` to add a granular "outline tool."
2. **System Prompt Calibration:** It updates the system prompt to instruct the LLM on when to use the new tool.
3. **Data Granularity:** It moves the system from "High-Level Search" to "Detailed Lesson Extraction," illustrating how agentic tools can be nested to provide deeper data insights.

## Key Takeaways

- **Plan Mode is non-negotiable for multi-file changes.** Direct execution works for single-file edits. Anything touching 2+ files needs the reasoning/actuation separation.
- **MCP Playwright closes the feedback loop.** The agent can validate its own changes in a live environment -- this is the primitive that turns code generation into systems engineering.
- **The memory hierarchy is your governance layer.** Use `CLAUDE.md` for team constraints, `CLAUDE.local.md` for your environment, and global for your personal standards.
- **`@` tagging > hoping the agent finds the right files.** Surgical context injection beats agentic search when you already know the relevant modules.
