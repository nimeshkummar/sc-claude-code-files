# Quiz: Web App from Figma Mockup

## Questions

### Q1: The two-MCP pipeline

This lesson uses two MCP servers together. Name them, describe what role each plays (one is an "Eye," one is a "Hand"), and explain how they compose into a single pipeline.

### Q2: Component hierarchy source

The agent generates a React component hierarchy: Sidebar, Dashboard, KPICard. Why is that specific hierarchy chosen -- is it a Next.js convention, a Claude best practice, or something else?

### Q3: Agentic API discovery

The static dashboard works, but the data is hardcoded. The user says "use real FRED data." The agent has never seen the FRED API before and has no documentation. What does the agent do, and what's the one thing only the user can provide?

### Q4: MCP scope

You add the Figma MCP server to this project via `/mcp`. Your teammate opens a different repo and tries to use the Figma MCP. It's not available. Why?

### Q5: End-to-end sequence

Walk through the full end-to-end sequence of this lesson -- from the very first input to the final running application. How many distinct phases are there?

---

## Answers

### A1

- **Figma MCP** ("Eye"): Extracts layout, CSS, and component hierarchy from the static mockup
- **Playwright MCP** ("Hand"): Renders the code in a browser and verifies visual fidelity

They compose into a **closed feedback loop**, not a one-shot pipeline:

```
Figma MCP (design source) → Agent generates code → Playwright MCP (visual verification)
     ↑                                                        ↓
     └──────────── Compare: does output match input? ─────────┘
```

If the Playwright screenshot doesn't match the Figma mock, the agent reads the discrepancy, patches the CSS, and re-verifies.

### A2

The hierarchy is **derived from the Figma layer structure**. Not a Next.js convention, not a Claude best practice -- the agent uses `Get Code` and `Get Design Rules` to extract the layer hierarchy from Figma and scaffolds components that match.

Sidebar exists as a component because it exists as a layer in Figma. This is **Design-Driven Development** -- the design is the source of truth for code structure.

### A3

**Agentic API discovery.** The agent:
1. Searches the web for "Federal Reserve Economic Data API"
2. Identifies the need for an API key
3. Locates endpoints for specific series (e.g., `UNRATE` for unemployment)
4. Writes the service layer and updates components to consume async data

The only thing the user provides is the **API key** -- a credential the agent can't and shouldn't generate. Everything else (discovery, endpoint mapping, implementation, component wiring) is autonomous.

### A4

MCP connections are **project-scoped, not global**. Each project defines its own MCP servers via `/mcp`. The teammate's repo has no Figma MCP because it was never added there.

Same pattern as the memory hierarchy: `CLAUDE.md` (project) vs `~/.claude/CLAUDE.md` (global). Project-level keeps configurations isolated and intentional.

### A5

Six distinct phases:

1. **Figma mockup provided** (design spec)
2. **Agent extracted layout and CSS** (ingestion -- `Get Image`, `Get Code`, `Get Design Rules`)
3. **Agent scaffolded Next.js components** (implementation)
4. **Playwright verified visual fidelity** (validation)
5. **Agent discovered and integrated FRED API** (data layer)
6. **Live dashboard running on `localhost:3000`** (result)

Key distinction: ingestion (reading the design) and implementation (writing the code) are separate phases. The agent first *reads*, then *writes*.
