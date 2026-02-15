# Quiz: Parallel Agents and Custom Commands

## Questions

### Q1: Eliminating prompt repetition

You find yourself typing the same prompt every time you start a new feature: "Implement this feature, write tests, follow existing patterns, document changes in `frontend-changes.md`." What's the mechanism to eliminate this repetition, where does the file live, and what are the two scope levels?

### Q2: The single-threaded problem

You need to build three independent features simultaneously: dark mode UI, FastAPI tests, and linting setup. Why can't you just run three Claude instances in the same directory, and what's the solution?

### Q3: Semantic conflict resolution

Your three worktree agents have finished. The testing agent added `pytest` to `pyproject.toml` and the quality agent added `black` to the same file. You now have a merge conflict. What do you do, and why is the agent's approach better than manual editing?

### Q4: Worktree cleanup

After merging all three worktrees back to main, what's the cleanup step and why is it important?

### Q5: The bottleneck shift

What's the fundamental bottleneck shift that worktrees create? Before worktrees, what limited your throughput? After worktrees, what's the new bottleneck?

---

## Answers

### A1

Create a `.claude/commands/implement-feature.md` file. It becomes the `/implement-feature` slash command. Use `$ARGUMENTS` for dynamic input injection.

Two scope levels:
- **Project-level** (`.claude/commands/`): Committed to Git, shared with the team
- **Global** (`~/.claude/commands/`): Local to your machine, applies to all projects

### A2

Three agents in the same directory will overwrite each other's files -- they're all writing to the same working directory.

**Solution: Git Worktrees.** Create a `.trees/` directory and spawn isolated environments:
```bash
git worktree add .trees/ui_feature -b ui_feature
git worktree add .trees/testing_feature -b testing_feature
git worktree add .trees/quality_feature -b quality_feature
```

Each worktree is an additional working directory linked to the same `.git` database but checked out to a different branch. Unlike `git clone` (which duplicates all history), worktrees share history and merge locally. Each Claude instance runs in its own worktree -- isolated files, no overwrites.

### A3

Ask Claude to "merge all worktrees and fix conflicts."

The agent's approach is superior because it resolves conflicts by **semantic intent**, not text-level diffing. It understands that adding `pytest` and adding `black` are both additive, non-conflicting in purpose -- so it unifies them into a valid config file. Manual editing with `<<<<HEAD` markers forces you to make text decisions without reasoning about what each change *meant to do*.

### A4

Delete the worktrees:
```bash
git worktree remove .trees/ui_feature
git worktree remove .trees/testing_feature
git worktree remove .trees/quality_feature
```

Why it matters:
1. **Branch locks** -- Git won't let you check out a branch that's already checked out in another worktree. Stale worktrees block future branch operations.
2. **Disposability** -- Worktrees are designed to be temporary. Create for the task, merge, delete. Main returns to a stable, feature-complete state.
3. **Agent confusion** -- Future Claude sessions might discover `.trees/` and treat stale worktrees as active workstreams.

### A5

- **Before worktrees:** The bottleneck is agent speed -- you wait for one agent to finish before starting the next task. Sequential execution.
- **After worktrees:** The bottleneck shifts to your ability to **scope and parallelize work** -- can you decompose features into independent streams that don't conflict?

The agents are fast. The constraint is now whether you can break work into clean, non-overlapping streams. That's an architectural skill, not a coding skill.
