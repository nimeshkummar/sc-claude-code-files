# Refactoring Notebooks into Dashboards

This breakdown analyzes the Legacy Refactoring and Application Migration capabilities of Claude Code. It demonstrates how to transition from "Exploratory Scripting" (Jupyter Notebooks) to "Productionized Dashboards" (Streamlit) using agentic workflows.

## Core Architectural Concepts

1. **Automated Decoupling (Separation of Concerns):** Moving from a monolithic notebook -- where data extraction, business logic, and rendering are intertwined -- to a modular architecture (`data_loader.py`, `metrics.py`).
2. **Spec-Driven Migration:** Using markdown spec files (`convert-to-dashboard.md`) to define the layout and behavioral constraints of the target application before generation.
3. **Rapid Application Development (RAD):** Using Streamlit to convert Python logic into an interactive web interface without writing frontend HTML/CSS/JS.
4. **Iterative UX Refinement:** The feedback loop of generating a base dashboard, identifying usability gaps, and patching them via natural language prompts.

---

## 1. The Problem: Monolithic Technical Debt

The starting point is a legacy artifact: a Jupyter Notebook analyzing e-commerce data.

- **State:** The notebook is functional but messy. It mixes `pandas` data loading with cell-based visualization logic.
- **Pain Point:** Extracting Year-Over-Year (YoY) revenue insights is tedious. Visualizations are static, and logic is trapped in cells -- hard to reuse or test.
- **Goal:** Refactor into a clean codebase without losing the exploratory insights.

### Why notebooks become technical debt

| Strength | Weakness |
|----------|----------|
| Fast exploration | Logic trapped in cell execution order |
| Inline visualization | Can't unit test individual functions |
| Low barrier to start | Mixes ETL, business logic, and rendering in one file |
| Great for prototyping | Can't ship to stakeholders as a product |

## 2. The Refactor: Architecting the Modules

Instead of asking Claude to "fix the notebook," the approach is to provide a structured prompt via a markdown spec file that defines the target architecture.

- **The Spec:** Explicitly requests Separation of Concerns:
  - **Module A (`data_loader.py`):** Data Loading & Processing (ETL)
  - **Module B (`metrics.py`):** Metric Calculations (Business Logic)
  - **Artifacts:** A clean notebook consuming these modules, plus `requirements.txt`

- **Agent Execution:** Claude uses the `NotebookRead` tool to parse the `.ipynb` file, understands the data structure, and generates the `.py` files using Object-Oriented patterns.

- **Result:** Logic becomes testable Python code. The notebook becomes merely a presentation layer importing from the modules.

### The modular architecture

```
Before:                          After:
notebook.ipynb                   data_loader.py    (ETL)
  ├── pandas loading               ↓
  ├── metric calculations        metrics.py        (Business Logic)
  ├── visualizations               ↓
  └── everything intertwined    notebook.ipynb     (Presentation only)
                                   ↓
                                dashboard.py       (Streamlit app)
```

## 3. Context-Aware Debugging

During the refactor, a `KeyError` occurs in a specific cell.

- **Workflow:** Paste the specific error and the cell code.
- **Resolution:** Claude analyzes the new modular structure, identifies the data mismatch in the dictionary key, and patches the specific function.
- **Key point:** The agent can debug its own generated code immediately within the same session. The modular structure makes the bug surface clearly -- if everything were still in one notebook, the root cause would be buried.

## 4. The Migration: From Script to App

The next step: move from a static notebook to a dynamic web app using Streamlit.

- **The Spec (`convert-to-dashboard.md`):** Defines the UI layout strictly:
  - **Header:** Title + Filter
  - **Top Row:** KPI Cards (Revenue, Growth, AOV)
  - **Middle Row:** Charts (Revenue by Category/State)

- **Execution:** Claude generates `dashboard.py`. It reuses the previously created logic modules (`metrics.py`), proving the value of the earlier refactor. The separation of concerns pays off -- the dashboard imports business logic rather than reimplementing it.

- **Deployment:**
  ```bash
  pip install streamlit
  streamlit run dashboard.py
  ```

### Why the earlier refactor matters here

Without the modular refactor, migrating to Streamlit would require rewriting the business logic. Because `metrics.py` already exists as a standalone module, the dashboard is purely a UI layer that imports and displays. Two spec files, two steps:

1. `refactor-notebook.md` → modular Python
2. `convert-to-dashboard.md` → Streamlit UI consuming those modules

## 5. UX Iteration and Polish

The initial dashboard works but lacks usability.

- **Issues:** Defaults to 2024 (where data is incomplete/empty), showing blank charts.
- **Fixes via natural language prompts:**
  1. **Set Default State:** Hardcode start year to 2023
  2. **Granularity:** Add a Month filter
  3. **Cleanup:** Remove empty KPI cards

- **Final State:** A fully interactive dashboard where business logic is decoupled from the UI, allowing easy updates to either layer independently.

### The UX iteration loop

```
Generate base dashboard → Run it → Identify usability gap → Describe fix in natural language → Agent patches → Repeat
```

This loop is fast because each fix is a targeted prompt, not a re-architecture. The modular structure means UX changes only touch `dashboard.py`, never the business logic.

## Key Takeaways

- **Notebooks are prototypes, not products.** The refactor pattern extracts testable, reusable modules from exploratory code. The notebook becomes a thin presentation layer.
- **Two spec files, two migrations.** First spec decouples logic into modules. Second spec defines the dashboard layout. Each is a discrete, reviewable step.
- **Separation of concerns pays compound interest.** The modular refactor made the Streamlit migration trivial -- `dashboard.py` imports from `metrics.py` instead of reimplementing calculations.
- **UX iteration is natural language patching.** Once the base dashboard exists, fixes like "default to 2023" or "add a month filter" are one-prompt changes.
- **The transformation: read-only artifact to shippable tool.** A notebook only you can read becomes a dashboard stakeholders can interact with.
