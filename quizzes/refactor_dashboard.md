# Quiz: Refactoring Notebooks into Dashboards

## Questions

### Q1: The wrong approach

You have a Jupyter Notebook with 200 cells that mixes data loading, metric calculations, and visualizations. A colleague says "just ask Claude to clean up the notebook." Why is that the wrong approach, and what do you do instead?

### Q2: Why the refactor enables migration

After Claude refactors your notebook into modules, you decide to build a Streamlit dashboard. Claude generates `dashboard.py` and it works on the first run. Why was the earlier refactor critical to making this migration easy? What would have happened without it?

### Q3: UX iteration pattern

Your generated dashboard loads, but it defaults to 2024 where data is incomplete -- all charts are blank. Describe the iteration pattern you use to fix this and why it's fast.

### Q4: Modular debugging advantage

During the refactor, a `KeyError` occurs in a cell. Why does the modular architecture make this bug easier to find than if everything were still in the original notebook?

### Q5: End-to-end transformation

What's the end-to-end transformation this workflow achieves? What did you start with, what did you end with, and why does the end state unlock value that the starting state didn't?

---

## Answers

### A1

"Clean up the notebook" preserves the monolithic structure -- you still end up with ETL, business logic, and rendering intertwined in cells. A cleaner notebook is still untestable, unreusable, and unshippable.

Instead, write a spec file requesting **Separation of Concerns**:
- **Module A (`data_loader.py`):** ETL
- **Module B (`metrics.py`):** Business logic
- **Artifact:** A thin notebook that imports from these modules

The notebook becomes a presentation layer only. Logic becomes testable Python code.

### A2

`dashboard.py` imports from `metrics.py` -- it's a pure UI layer with zero reimplementation. Separation of concerns pays **compound interest**.

Without the refactor, Claude would have to rewrite the business logic inside `dashboard.py`, creating two copies of the same calculations. Every future change (new metric, formula fix) would need to be made in two places. That's how bugs breed.

### A3

The UX iteration loop:
```
Generate base dashboard → Run it → Identify usability gap → Describe fix in natural language → Agent patches → Repeat
```

It's fast because each fix is a **targeted prompt**, not a re-architecture. The modular structure means UX changes only touch `dashboard.py`, never the business logic. Specific fixes like "default to 2023" or "add a month filter" are one-prompt changes -- no spec file needed, no Plan Mode.

### A4

In the original monolithic notebook, a `KeyError` could originate from any of 200 cells. Data loading, transformation, metric calculation, and rendering are all tangled -- you'd have to trace through the entire execution chain.

In the modular structure, the bug surfaces at the **module boundary**. You know it's either in `data_loader.py` (data shape) or `metrics.py` (key access). The error message tells you which function, which module, which key -- because each module has a single responsibility.

The agent can also debug its own generated code immediately within the same session, since it has full context of what it just created.

### A5

**Started with:** A read-only artifact -- a Jupyter Notebook that only you can run and interpret. Static, untestable, trapped in cells.

**Ended with:** A shippable tool -- a Streamlit dashboard that stakeholders can interact with. Modular, testable, decoupled.

**The value unlock:** The notebook kept insights locked to the person who wrote it. The dashboard unlocks the data for non-technical users. A product manager can filter by year and month, see KPI cards, and draw their own conclusions without asking you to re-run cells.

The transformation is one pipeline with two spec files:
1. `refactor-notebook.md` → modular Python
2. `convert-to-dashboard.md` → Streamlit UI consuming those modules
