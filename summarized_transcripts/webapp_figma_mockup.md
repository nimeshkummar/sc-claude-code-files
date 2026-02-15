# Web App from Figma Mockup

This breakdown analyzes the Design-to-Code Pipeline and API Integration capabilities of Claude Code. It demonstrates the transition from a visual artifact (Figma) to a functional, data-driven application (Next.js) using agentic orchestration.

## Core Architectural Concepts

1. **Multimodal MCP Chain:** The workflow links two distinct MCP servers:
   - **Figma MCP:** Acts as the "Eye" for design -- extracts layout, CSS, and component hierarchy from a static mockup.
   - **Playwright MCP:** Acts as the "Hand" for validation -- renders the code in a browser and verifies visual fidelity against the original design.

2. **Design-Driven Development (DDD):** The code structure is derived directly from the Figma layer structure using `Get Code` and `Get Design Rules` tools. The React component hierarchy mirrors the Figma layers -- not arbitrary, but design-sourced.

3. **Agentic API Discovery:** The agent moves beyond static mocks by autonomously researching external APIs, understanding authentication requirements, and implementing the data fetching layer without explicit documentation from the user.

---

## 1. Setup and Infrastructure

The lesson begins by initializing a modern frontend stack.

- **Stack:** Next.js (React framework)
- **Goal:** Build a dashboard visualizing Federal Reserve Economic Data (FRED)
- **Prerequisites:**
  - **Figma Account:** Required to access the mockup via API
  - **MCP Server Initialization:** Add Figma and Playwright MCP servers to the current project scope via `/mcp` command

### MCP scope

MCP connections are **project-specific, not global**. Each project defines its own MCP servers. This means:

| Scope | What it means |
|-------|---------------|
| Project MCP | Only available in the current repo |
| Global MCP | Would apply everywhere (not how this works) |

This is analogous to the `CLAUDE.md` vs `~/.claude/CLAUDE.md` distinction -- project-level keeps configurations isolated.

## 2. The Ingestion Phase: From Pixel to Code

The agent converts the Figma mockup into a functional React application.

- **The Artifact:** A Figma mockup of an economic dashboard

- **Mechanism:**
  1. **Enable Dev Mode:** Enable the MCP server in Figma preferences
  2. **Layer ID:** Copy the specific Layer URL/ID to guide the agent to the right component

- **Tool Usage:**
  - `Get Image`: Retrieves a visual snapshot of the mockup
  - `Get Code`: Extracts CSS properties (flexbox, grid, colors) and DOM structure
  - `Get Design Rules`: Pulls design tokens and constraints

- **Implementation:** The agent scaffolds the `app/` directory in Next.js, creating components that match the visual hierarchy:

```
app/
├── components/
│   ├── Sidebar.tsx
│   ├── Dashboard.tsx
│   └── KPICard.tsx
├── page.tsx
└── layout.tsx
```

The component hierarchy mirrors the Figma layer structure -- Sidebar, Main Dashboard, KPI Cards -- because the agent derived it from the design, not from convention.

## 3. The Validation Loop: Playwright MCP

Once the code is written, the agent verifies it against the original design.

1. **Navigate:** Playwright visits `localhost:3000`
2. **Screenshot:** Takes a screenshot of the running application
3. **Compare:** Agent (or user) compares the Playwright screenshot against the original Figma mock

This is the same Playwright validation loop from the Adding Features lesson, but applied to design fidelity instead of feature verification. The pattern is consistent: **agent validates its own output in a live environment.**

### The two-MCP feedback loop

```
Figma MCP (design source) → Agent generates code → Playwright MCP (visual verification)
     ↑                                                        ↓
     └──────────── Compare: does output match input? ─────────┘
```

If the output doesn't match, the agent can read the Playwright screenshot, identify discrepancies, and patch the CSS -- closing the loop without manual intervention.

## 4. The Data Layer: Real-World Integration

The static UI is functional, but the data is fake. The next step: connect to real data.

- **Agentic Research:** The user asks to populate charts with FRED data (CPI, Unemployment). The agent:
  1. Searches the web for "Federal Reserve Economic Data API"
  2. Identifies the need for an API key
  3. Locates endpoints for specific series (e.g., `UNRATE` for unemployment)

- **Actuation:**
  1. User provides the API key
  2. Agent writes a `service` layer to fetch data from the FRED API
  3. Updates frontend components to consume async data instead of hardcoded arrays

### The layered architecture

```
Figma (Design)  →  Components (UI)  →  Services (API Fetch)  →  FRED API (Data)
```

Each layer is independent. You can swap the data source (FRED → internal API) without touching the components, or redesign the UI without touching the service layer. Separation of concerns again.

## 5. Final State: Production-Ready Prototype

The final application is a live dashboard showing real-time economic indicators.

- **Outcome:** Static image → live, data-connected application in a single session
- **Traceability:** The dashboard includes links back to the data source -- a feature the agent added to ensure data provenance
- **What happened in sequence:**
  1. Figma mockup provided (design spec)
  2. Agent extracted layout and CSS (ingestion)
  3. Agent scaffolded Next.js components (implementation)
  4. Playwright verified visual fidelity (validation)
  5. Agent discovered and integrated FRED API (data layer)
  6. Live dashboard running on `localhost:3000` (result)

## Key Takeaways

- **Two MCP servers compose into a design-to-validation pipeline.** Figma MCP provides the design source; Playwright MCP verifies the output. The agent closes the loop between design intent and rendered reality.
- **Design-Driven Development means the Figma layers become the component hierarchy.** The code structure isn't invented -- it's derived from the design artifact.
- **Agentic API discovery eliminates the documentation tax.** The agent researches APIs, finds endpoints, and understands auth requirements autonomously. You provide the API key; it handles the rest.
- **MCP connections are project-scoped.** Add them per-project via `/mcp`, not globally. This keeps tool configurations isolated and intentional.
- **The architect provides the blueprint (Figma) and the data strategy (which API). The agent handles translation (Figma → CSS) and plumbing (API → component state).**
