# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Frequently used commands

| Task | Command | Description |
|------|---------|-------------|
| **Set up the environment** | `python -m venv venv && venv\Scripts\activate && pip install -r requirement.txt` | Creates a fresh virtual environment and installs all dependencies listed in `requirement.txt`. |
| **Activate the environment (Windows)** | `venv\Scripts\activate` | Activates the previously created virtual environment. |
| **Activate the environment (Unix/macOS)** | `source venv/bin/activate` | Same as above for POSIX shells. |
| **Run notebooks** | `jupyter lab` | Starts Jupyter Lab in the repository root so you can open any `.ipynb` file (e.g., `prompt_chaining_workflow.ipynb`). |
| **Execute a single notebook cell from the CLI** | `jupyter nbconvert --to notebook --execute --inplace <notebook>.ipynb` | Re‑runs the notebook in‑place – useful for quick verification. |
| **Lint / format** | `ruff check .` | Runs the `ruff` linter on the whole repository. |
| **Auto‑format** | `ruff format .` | Applies `ruff`'s auto‑formatter. |
| **Run tests (if pytest tests are added)** | `pytest` | Discovers and runs all tests under `tests/`. |
| **Run a single test file** | `pytest tests/test_my_module.py` | Replace the path with the desired test file. |
| **Show help for any command** | `<command> --help` | Most CLI tools support `--help` for usage details. |

> **Note:** The repository currently does not contain a test suite, but the `pytest` commands are included for future test addition.

---

## High‑level architecture & key concepts

1. **LangGraph‑centric workflow** – All notebooks revolve around the **LangGraph** library (`langgraph.graph.StateGraph`). The typical pattern is:
   - Define a **TypedDict** (or Pydantic model) representing the mutable **state** that flows through the graph.
   - Implement **node functions** that accept the state, perform a single responsibility (e.g., call an LLM, invoke a tool, evaluate content), mutate the state, and return it.
   - Wire nodes together with **edges** (`START`, `END`, conditional edges) to form a directed execution graph.
   - **Compile** the graph into a **workflow** (`graph.compile()`) and invoke it with an initial state.

2. **Prompt‑chaining example** – `prompt_chaining_workflow.ipynb` demonstrates a concrete pipeline:
   - `create_outline` → generates a blog outline from a title.
   - `write_content` → expands the outline into a full article.
   - `evaluate_content` → asks the same LLM to rank and comment on the article.
   - The final state contains `outline`, `content`, and `feedback`.
   - This notebook serves as a template for any multi‑step LLM orchestration.

3. **Supporting notes** – The `Notes/` directory holds markdown files that explain LangGraph’s **four core concepts** (Graph, State, Nodes, Edges) and best‑practice design guidelines. Reviewing `Notes/Langgraph_core_concepts.md` gives a concise mental model for building new graphs.

4. **Environment handling** – The notebooks load environment variables via `dotenv.load_dotenv()`. Keep any required keys (e.g., API tokens) in a local `.env` file; the `.gitignore` already excludes it.

5. **Virtual environment** – All Python dependencies, including `langgraph`, `langchain_groq`, `ruff`, and Jupyter, are installed into the repository‑scoped `venv`. This isolates the project from global packages.

---

## Development workflow tips (concise)

- **Iterate quickly:** Edit a notebook cell, run it, then re‑compile the graph (`graph.compile()`) to see updated behavior.
- **State inspection:** After `workflow.invoke(initial_state)`, print the entire state or specific keys to verify intermediate results.
- **Add new nodes:** Follow the existing pattern – define a function returning the same TypedDict, register it with `graph.add_node`, and connect edges.
- **Conditional edges:** Use `graph.add_conditional_edges` (or the `add_edge` overload that accepts a condition lambda) when branching based on state values.
- **Testing:** When tests are added, place them under a `tests/` folder and import the graph components directly (e.g., `from prompt_chaining_workflow import create_outline`). Use `pytest` fixtures to spin up a minimal state.

---

## Cursor / Copilot rules (if present)

No `.cursor` or `.github/copilot‑instructions.md` files were detected in the repository, so there are no special AI‑assistant rules to enforce.

---

*End of CLAUDE.md*