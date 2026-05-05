# LangGraph Structured Learning

**A personal, self‑contained learning resource for mastering LangGraph – the stateful workflow orchestration framework for LLM‑powered applications.**

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Project Structure](#project-structure)
4. [Prerequisites](#prerequisites)
5. [Installation](#installation)
6. [Running the Notebooks](#running-the-notebooks)
7. [Key Concepts & Resources](#key-concepts--resources)
8. [Contributing](#contributing)
9. [License](#license)
10. [Acknowledgements](#acknowledgements)

---

## Overview

This repository is a **structured learning playground** for LangGraph. It contains:

* **Hands‑on Jupyter notebooks** that implement end‑to‑end LangGraph workflows (prompt‑chaining, state management, tool calling, multi‑step reasoning).
* **Markdown notes** that capture core concepts, design best practices, and deep‑dive explanations of LangGraph’s four fundamental ideas (Graph, State, Nodes, Edges).
* A **Python virtual environment** with all required dependencies, allowing you to run the examples out‑of‑the‑box.

The purpose is to provide a concrete, reproducible reference while you explore LangGraph’s capabilities and build production‑grade AI agents, RAG pipelines, and multi‑agent systems.

---

## Features

* **Stateful graph execution** – Demonstrates how to define a shared `state` object and pass it through a directed graph of nodes.
* **Prompt chaining** – Shows how to chain LLM calls (outline generation → content creation → evaluation) without manual data copying.
* **Evaluation loop** – Uses the same LLM to critique its own output, illustrating iterative refinement.
* **Comprehensive documentation** – In‑depth markdown files (`Langgraph_core_concepts.md`, etc.) explain theory, compare LangGraph to linear chains, and list design guidelines.
* **Ready‑to‑run notebooks** – `LLM_workflow.ipynb`, `prompt_chaining_workflow.ipynb`, and `workflow_1.ipynb` cover a range of scenarios from simple state graphs to more complex agentic pipelines.

---

## Project Structure

```
.
├─ .gitignore                # Standard Python & tooling ignores (+ .claude folder)
├─ .claude/                  # Internal Claude‑Code files (auto‑generated)
├─ LICENSE
├─ README.md                 # ⇡ You are here
├─ requirement.txt           # Pin‑list of Python packages
├─ LLM_workflow.ipynb        # Basic LangGraph workflow example
├─ prompt_chaining_workflow.ipynb  # Prompt‑chaining blog‑post pipeline
├─ workflow_1.ipynb          # Additional workflow demonstration
├─ Notes/
│   ├─ Langgraph_core_concepts.md      # Core concept deep‑dive
│   ├─ LangGraph_Four_Core_Concepts_Deep_Dive_for_Beginners.md
│   └─ test.ipynb
└─ venv/                     # Python virtual environment (generated via `python -m venv venv`)
```

---

## Prerequisites

* **Python 3.11** (or newer)
* **Git** – to clone the repository
* **Jupyter** – installed via the `requirement.txt` or via `pip install jupyterlab`

---

## Installation

```bash
# 1️⃣ Clone the repo
git clone https://github.com/your-username/langgraph-structured-learning
cd langgraph-structured-learning

# 2️⃣ Create and activate the virtual environment
python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

# 3️⃣ Install dependencies
pip install -r requirement.txt
```

> **Note:** The repository already contains a populated `venv/` directory for convenience, but creating a fresh environment as shown above guarantees a clean, reproducible setup.

---

## Running the Notebooks

```bash
# Launch Jupyter Lab (or Notebook)
jupyter lab
```

* Open `prompt_chaining_workflow.ipynb` to see a complete **title → outline → content → evaluation** pipeline powered by a Groq LLM (`llama‑3.3‑70b‑versatile`).
* Execute cells sequentially; the final state is printed at the end of the notebook and includes the generated blog outline, full article, and a structured feedback rating (1‑5) with improvement suggestions.

The notebooks are self‑documenting—each cell includes a short description of its purpose and the underlying LangGraph constructs (`StateGraph`, `START`, `END`, node registration, edge definition, compilation, and invocation).

---

## Key Concepts & Resources

* **LangGraph** – Stateful graph orchestration built on top of LangChain.
  * Official docs: https://langchain-ai.github.io/langgraph/
* **Core concepts** – See `Notes/Langgraph_core_concepts.md` for a concise walkthrough of Graph, State, Nodes, and Edges, plus best‑practice guidelines.
* **Prompt‑chaining pattern** – Demonstrated in `prompt_chaining_workflow.ipynb`.
* **State schema** – Defined using `TypedDict` for static type checking and clear documentation of expected fields.

---

## Contributing

This repository is primarily a personal learning space, but contributions are welcome:

1. Fork the repo and create a feature branch.
2. Follow the existing code style (PEP 8, type‑annotated `TypedDict` where appropriate).
3. Add or update notes in the `Notes/` directory for any new concepts explored.
4. Submit a Pull Request with a clear description of the change and, if applicable, an updated notebook that demonstrates the new functionality.

---

## License

This project is licensed under the **MIT License** – see the `LICENSE` file for details.

---

## Acknowledgements

* **LangGraph** and the LangChain ecosystem for providing the underlying framework.
* **Groq** for the hosted LLM used in the examples.
* The many open‑source Python libraries listed in `requirement.txt` that make rapid prototyping possible.

---

*Happy learning, and enjoy building stateful AI workflows with LangGraph!*
This repository is my personal structured learning resource for LangGraph. I use this space to document concepts, notes, experiments, and hands-on implementations while building stateful and agentic AI systems.  The goal is to deeply understand LangGraph by learning step-by-step — from fundamentals to real-world agent workflows.
