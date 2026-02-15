# Parallel Agents and Custom Commands

This breakdown analyzes Parallel Agentic Workflows and Agentic Extensibility. It moves beyond "using the tool" to "architecting the environment" for high-velocity AI development.

## 1. Agentic Extensibility: Custom Slash Commands

The transcript introduces a method to program the agent's interface itself, transforming repetitive prompts into executable system commands.

- **The Mechanism:** Instead of pasting the same "Implement this feature with these constraints" prompt repeatedly, you define it as a persistent tool.

- **File Structure:** Create a `.claude/commands/` directory and add markdown files (e.g., `implement-feature.md`). Each file becomes a slash command invoked as `/implement-feature`.

- **Argument Injection:** The `$ARGUMENTS` variable allows dynamic input injection. This is prompt templating at the filesystem level.

- **Architectural Value:** By standardizing how features are implemented (e.g., "always write changes to `frontend-changes.md`"), every agentic session adheres to the same process standards without manual enforcement.

### Example: Custom command file

```
# .claude/commands/implement-feature.md

Implement the following feature: $ARGUMENTS

Constraints:
- Write all UI changes to frontend-changes.md before modifying components
- Follow existing patterns in the codebase
- Write tests for any new public functions
```

### Scope levels for custom commands

| Directory | Scope | Shared via Git |
|-----------|-------|----------------|
| `.claude/commands/` | Project-level | Yes |
| `~/.claude/commands/` | Global (all projects) | No |

## 2. Parallel Agentic Workflows: Git Worktrees

The core problem: coding agents are single-threaded. If you run three agents on the same folder, they overwrite each other's files.

- **The Solution: Git Worktrees.** Unlike cloning a repo (which duplicates the entire history), a worktree creates a new working directory linked to the *same* git history but checked out to a different branch.

- **Implementation:**
  1. Create a `.trees/` directory to house temporary worktrees.
  2. Spawn isolated environments on separate branches:
     ```bash
     git worktree add .trees/ui_feature -b ui_feature
     git worktree add .trees/testing_feature -b testing_feature
     git worktree add .trees/quality_feature -b quality_feature
     ```
  3. Open three terminals, each running a Claude instance in its own worktree. They work in parallel without file conflicts.

- **The mental model:** Each worktree is an isolated "engineer" with its own working directory but shared git history. You direct three streams of work simultaneously instead of sequencing them.

### Worktree vs. Clone vs. Branch

| Approach | Disk cost | Shared history | Parallel agents | Merge path |
|----------|-----------|----------------|-----------------|------------|
| Same folder | None | Yes | No -- file conflicts | N/A |
| `git clone` | Full duplicate | Independent | Yes | Remote push/pull |
| `git worktree` | Working dir only | Yes (same `.git`) | Yes | Local merge |

## 3. Agentic Integration: Automated Conflict Resolution

Parallelism inevitably leads to collision. The transcript demonstrates handling this agentically.

- **The Conflict:** Both the "Testing" agent and the "Quality" agent modified `pyproject.toml` to add dependencies, creating a merge conflict.

- **The Resolution:** Instead of manually editing `<<<<HEAD` markers, ask Claude to "merge all worktrees and fix conflicts."

- **Logic:** The agent analyzes the *semantic intent* of both changes (adding `pytest` vs. adding `black`) and unifies them into a valid configuration file. It understands that both changes are additive and non-conflicting in purpose.

- **Cleanup:** Commit the merge, then delete the temporary worktrees:
  ```bash
  git worktree remove .trees/ui_feature
  git worktree remove .trees/testing_feature
  git worktree remove .trees/quality_feature
  ```
  The main branch returns to a stable, feature-complete state.

## Summary of Primitives

| Primitive | Purpose |
|-----------|---------|
| `$ARGUMENTS` | Variable in custom command templates that accepts user input |
| `.claude/commands/` | Directory for project-level custom slash commands |
| `.trees/` | Directory convention for temporary worktrees |
| `git worktree add` | The "fork" operation for your local environment, enabling parallel agent streams |
| `/implement-feature` | Example custom command that standardizes the coding lifecycle |

## Key Takeaways

- **Custom commands eliminate prompt repetition.** If you've typed the same constraints three times, it belongs in a `.claude/commands/` file.
- **Git worktrees unlock parallelism.** Three agents, three worktrees, three features at once. The bottleneck shifts from agent speed to your ability to define and direct the work.
- **Let the agent resolve merge conflicts semantically.** It understands that adding `pytest` and adding `black` are both valid -- it merges intent, not just text.
- **Worktrees are disposable.** Create them for the task, merge the results, delete them. They're cheap because they share git history.
