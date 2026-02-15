# Quiz: Testing, Error Debugging, and Code Refactoring

## Questions

### Q1: The debugging protocol

Your RAG pipeline returns empty results. You suspect the config layer. A junior engineer would paste the error into Claude and say "fix this." What's the correct protocol, and what are the three phases of the debugging loop?

### Q2: Spec-driven refactoring

You need to refactor a backend service from single-turn to multi-turn tool use. It touches 4 files with clearly defined current and desired behavior. Do you use a chat prompt or a different approach? What goes in the artifact, and why is it superior?

### Q3: Sub-agent dispatch and role framing

You're choosing between two approaches for a system redesign. Instead of picking one on intuition, what do you ask Claude to do, and what role does this put you in?

### Q4: Multi-turn RAG architecture

A simple RAG handles "What is in Lesson 3?" but fails on "Compare Lesson 3 vs Lesson 5." What's the architectural limitation, and what does the refactored system do differently?

### Q5: The regression ratchet

After debugging a bug with the test-first protocol, what permanent artifact does the process leave behind, and why does it matter more than the fix itself?

---

## Answers

### A1

**Setup (before the loop):**
- **Constraint Injection** -- name the suspect files in the prompt to narrow the agent's search space
- **Extended Thinking** -- tell Claude to "think a lot" to trace error propagation paths before proposing anything
- **Write the test first** -- reproduction test before any fix attempt

**The Red-Green Loop:**
```
Run Tests (Fail) → Analyze Failure (Zero Results) → Fix Config (MAX_RESULTS=0) → Run Tests (Pass)
```

Key: the test comes before the fix, not after. Instrument first, then change code.

### A2

Use a spec file (`<feature>-refactor.md`) containing:
1. Current Behavior
2. Desired Behavior
3. Example Flows
4. Constraints

Superior to chat because:
- Separates design authority (yours) from implementation labor (the agent's)
- Persistent artifact -- survives `/clear`, stays in the repo as documentation
- Concrete data reduces hallucination

### A3

Ask Claude to dispatch two subagents to brainstorm options and present trade-offs.

- The **agent** becomes the Staff Engineer (researches and recommends)
- **You** become the Technical Lead (evaluates and decides)

The agent does the analysis work. You have the decision authority.

### A4

**Limitation:** Single-turn RAG does `Query → Search → Answer`. One search can't retrieve enough data from multiple sources for comparison queries.

**Refactored (multi-turn):**
```
Query → Search Tool 1 (outline) → LLM: "I need lesson details"
      → Search Tool 2 (details) → Final Answer
```

The `AIGenerator` uses an iterative loop that accumulates context across multiple tool calls. Each iteration, the model evaluates what it has and what it's missing, looping until the stop condition is met.

### A5

The test case. It serves as a **regression ratchet** -- quality only moves in one direction.

- The fix is temporary value (solves today's problem)
- The test is permanent value (the bug can never silently reappear)
