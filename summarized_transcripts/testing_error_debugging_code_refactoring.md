# Testing, Error Debugging, and Code Refactoring

This breakdown analyzes the System Hardening and Architectural Refactoring workflows demonstrated in the transcript. It shifts focus from "building features" to "engineering robustness" and "managing complexity."

## 1. The Debugging Protocol: Test-Driven Agentic Engineering

The transcript rejects the common pattern of "paste error, hope for fix." Instead, it introduces a methodical, test-first approach to fix a RAG pipeline failure.

- **Constraint Injection:** The prompt explicitly names the suspect files (`AIGenerator`, `rag_system.py`, `search_tools`). This narrows the agent's search space, preventing unrelated code changes.

- **Extended Thinking:** The user explicitly asks Claude to "think a lot." This allocates more compute to the reasoning chain before token generation, allowing the model to trace potential error propagation paths (configuration vs. logic vs. API) before proposing a fix.

- **Verification via Instrumentation:** The agent is tasked to write tests *first* (using `pytest` and ChromaDB mocks).

**The Red-Green Loop:**

```
Run Tests (Fail) → Analyze Failure (Zero Results) → Fix Config (MAX_RESULTS=0 found) → Run Tests (Pass)
```

**Architectural Value:** This leaves behind a regression suite, permanently increasing the system's "ratchet" of quality. The bug can never silently reappear.

## 2. Spec-Driven Refactoring (The "Design Doc" Pattern)

For complex changes, the transcript moves away from chat prompts entirely.

- **The Artifact (`backend-tool-refactor.md`):** A dedicated markdown file is created to define the refactor spec.

- **Contents:**
  - Current Behavior
  - Desired Behavior (multi-turn loops)
  - Example Flows
  - Constraints (e.g., "Verify external behavior, ignore internal state")

- **Why this matters:** The spec file is the architect's primary interface. You write the spec; the agent executes the implementation. This separates design authority from implementation labor.

### When to use the Design Doc pattern vs. chat prompts

| Scenario | Use Chat | Use Spec File |
|----------|----------|---------------|
| Single-file bug fix | Yes | No |
| Multi-file refactor | No | Yes |
| Behavior change with defined I/O | No | Yes |
| Exploratory prototyping | Yes | No |

## 3. Agentic Parallelism (The "Sub-Agent" Dispatch)

This is a critical advanced pattern. Instead of asking Claude to implement immediately, you ask it to "dispatch two subagents to brainstorm potential options."

**The Process:**

1. **Dispatch:** The agent spawns parallel reasoning threads to analyze the codebase.
2. **Trade-off Analysis:** It returns two distinct architectural approaches:
   - **Option A:** Iterative -- safer, simpler.
   - **Option B:** Recursive -- comprehensive, better for long-term maintainability.
3. **Human Decision:** You act as the Technical Lead, selecting based on current needs.

**The shift:** This transforms the AI from a "Code Generator" into a "Staff Engineer" that presents trade-offs for your executive decision. You don't write the code -- you make the architectural call.

## 4. Recursive Tool Use Architecture

The refactor itself addresses a fundamental limitation in simple RAG systems: Single-Turn vs. Multi-Turn Reasoning.

**Before (single-turn):**

```
Query → Search → Answer
```

Fails for comparison queries that require data from multiple sources.

**After (iterative/recursive):**

```
Query → Search Tool 1 (Course Outline) → LLM Analysis ("I need lesson details")
      → Search Tool 2 (Lesson Details) → Final Answer
```

**Mechanism:** The `AIGenerator` is refactored to support a loop that accumulates context across multiple tool calls until the stop condition is met. Each iteration adds information the model determined it was missing.

## Key Takeaways

- **Never debug with a raw error paste.** Force the agent to write the reproduction test case first. The test stays behind as a regression guard.
- **Use spec files for any refactor with defined inputs/outputs.** Write a `<feature>-refactor.md` with current behavior, desired behavior, example flows, and constraints. Feed it to Claude Code with `@`.
- **Dispatch subagents for architectural decisions.** When you're choosing between two approaches, don't pick one and hope. Ask the agent to present both with trade-offs, then you decide.
- **Multi-turn tool use is the unlock for complex RAG.** Single-turn search-and-answer breaks down for comparison or aggregation queries. The iterative loop pattern lets the agent determine what data it's missing and go get it.
