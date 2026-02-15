# Quiz: Adding Features with Claude Code

## Questions

### Q1: Multi-file feature implementation primitives

You're adding a new API endpoint requiring changes to the route handler, a service module, and the frontend component. You know which three files need to change. What two Claude Code primitives should you use before writing any code, and why does the order matter?

### Q2: Post-implementation verification

You've finished implementing a feature. What MCP integration lets you verify the implementation without manually opening a browser, and what three steps does it perform?

### Q3: Memory hierarchy layers

You have a team-wide rule ("All new tools must be registered in `search_tools.py`") and a local constraint (non-standard dev server port). Which memory layer does each go in, and what prevents the local rule from leaking to teammates?

### Q4: Context contamination

After 45 minutes on a complex refactor, you need to start a completely different feature. The agent keeps referencing decisions from the refactor. What's the problem, and what primitive fixes it?

### Q5: Plan Mode tradeoffs

A teammate argues Plan Mode is a waste of time. When are they right, and when are they wrong? What's the specific failure mode Plan Mode prevents?

---

## Answers

### A1

1. `@` tag the three files -- surgical context injection over letting the agent search blindly.
2. Plan Mode (`Shift+Tab` twice) -- separates reasoning from actuation for multi-file changes.

Order: `@` tag first to load correct context, then Plan Mode so the agent reasons over the right files. Planning without tagging risks a roadmap based on incomplete context.

### A2

Playwright MCP:
1. **Navigation** -- visits the live application URL
2. **Verification** -- takes screenshots to confirm the UI element rendered correctly
3. **Autonomous Testing** -- evaluates JavaScript or clicks elements to verify the frontend-backend handshake

This converts the agent from "Code Writer" to "Systems Tester" -- it closes the feedback loop by verifying its own implementation in a live environment.

### A3

- Team rule: `CLAUDE.md` (committed to Git, whole team sees it)
- Local constraint: `CLAUDE.local.md` (environment-specific)
- Prevention mechanism: `.gitignore` excludes `CLAUDE.local.md` from being committed

### A4

The problem is context contamination -- the window is polluted with prior logic causing "middle-of-the-prompt" degradation and conflicting instructions.

Fix: `/clear` to reset the slate, then `@` tag the relevant files for the new feature so the agent starts clean with only the right context.

### A5

- **Right:** Single-file edits. If the change is isolated to one file, direct execution is fine -- there's nothing to cascade into.
- **Wrong:** Any change touching 2+ files.
- **Failure mode:** The "cascading error" effect -- the agent makes a mistake in file A, builds on that mistake in file B, which compounds into file C. By the time you notice, the error is baked into multiple files.

Plan Mode catches bad architectural decisions at the plan stage, before any code is written.
