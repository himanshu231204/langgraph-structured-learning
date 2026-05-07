# LangGraph Structured Learning

**A comprehensive, self-contained learning repository for mastering LangGraph — a stateful workflow orchestration framework for building production-grade LLM-powered applications and multi-agent systems.**

> This repository provides hands-on implementation examples, in-depth conceptual documentation, and best-practice patterns for orchestrating complex AI workflows.

---

## 📑 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Running the Notebooks](#running-the-notebooks)
- [Workflow Examples](#workflow-examples)
- [Key Concepts](#key-concepts)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgements](#acknowledgements)

---

## Overview

This repository serves as a **structured learning playground** for LangGraph, containing:

- **Hands-on Jupyter notebooks** implementing end-to-end workflows: prompt-chaining, parallel execution, state management, tool calling, and multi-step reasoning pipelines
- **Comprehensive documentation** explaining LangGraph's core concepts (Graph, State, Nodes, Edges) with deep-dive explanations and design principles
- **Design patterns & best practices** for building scalable, maintainable AI workflows
- **Pre-configured environment** with all dependencies isolated in a Python virtual environment for reproducibility

Whether you're building AI agents, RAG pipelines, or multi-agent systems, this repository provides concrete, tested references to accelerate your development.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| **Stateful graph execution** | Demonstrates shared state management and node-to-node data flow through directed graphs |
| **Prompt chaining** | Illustrates sequential LLM calls (outline → content → evaluation) with automatic state passing |
| **Parallel workflows** | Shows concurrent node execution with delta-based state merging and aggregation strategies |
| **Structured outputs** | Uses Pydantic models with LLM structured output for type-safe API interactions |
| **Evaluation loops** | Implements iterative refinement patterns where LLMs critique and improve their own outputs |
| **State aggregation** | Demonstrates reducer functions and annotated types for combining parallel results |
| **Production patterns** | Covers error handling, retry logic, and workflow compilation for edge cases |

---

## 📂 Project Structure

```
langgraph-structured-learning/
├── README.md                                      # This file
├── LICENSE                                        # MIT License
├── CLAUDE.md                                      # Claude/Copilot instructions
├── REVISION_NOTES.md                              # Project revision history
├── requirement.txt                                # Python dependencies
│
├── 📓 Notebooks (Jupyter .ipynb files)
│   ├── LLM_workflow.ipynb                         # Basic LangGraph workflow
│   ├── prompt_chaining_workflow.ipynb             # Blog generation pipeline
│   ├── workflow_1.ipynb                           # Multi-step workflow example
│   │
│   └── parallel_workflow/
│       ├── simple_workflow.ipynb                  # Non-LLM parallel computation
│       └── llm_eassy_workflow.ipynb               # Parallel LLM evaluations
│
├── 📚 Notes/ (Conceptual documentation)
│   ├── Langgraph_core_concepts.md                 # Core concepts & theory
│   ├── LangGraph_Four_Core_Concepts_Deep_Dive_for_Beginners.md
│   ├── Parallel_Workflow_Revision.md              # Parallel patterns & diagrams
│   └── test.ipynb
│
└── venv/                                          # Python virtual environment
```

---

## 🚀 Quick Start

```bash
# Clone and setup
git clone https://github.com/your-username/langgraph-structured-learning
cd langgraph-structured-learning

# Create and activate virtual environment
python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

# Install dependencies
pip install -r requirement.txt

# Launch Jupyter
jupyter lab

# Open and run: prompt_chaining_workflow.ipynb
```

---

## 📋 Prerequisites

- **Python 3.11+** (tested with Python 3.11 and 3.12)
- **Git** (for cloning the repository)
- **pip** (Python package manager, included with Python 3.4+)
- **API credentials** (Groq API key for LLM examples; set in a `.env` file)

---

## 🔧 Installation

### Option 1: Fresh Environment (Recommended)

```bash
# 1. Clone the repository
git clone https://github.com/your-username/langgraph-structured-learning
cd langgraph-structured-learning

# 2. Create virtual environment
python -m venv venv

# 3. Activate virtual environment
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

# 4. Install dependencies
pip install -r requirement.txt

# 5. Set up environment variables
# Create a .env file in the project root and add:
# GROQ_API_KEY=your_groq_api_key_here
```

### Option 2: Using Existing venv/

The repository includes a pre-populated `venv/` directory. Simply activate and use:

```bash
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate
```

> **Note:** For maximum reproducibility and to ensure compatibility, creating a fresh environment using Option 1 is recommended.

---

## 📖 Running the Notebooks

### Launch Jupyter

```bash
jupyter lab
```

### Recommended Learning Path

1. **Start with:** `prompt_chaining_workflow.ipynb`
   - Demonstrates a complete **title → outline → content → feedback** pipeline
   - Shows fundamental LangGraph patterns: `StateGraph`, `START`, `END`, nodes, edges
   - Ideal introduction to prompt chaining

2. **Next:** `parallel_workflow/simple_workflow.ipynb`
   - Pure-Python example of parallel node execution
   - Illustrates delta-based state merging without LLM calls
   - Great for understanding concurrency patterns

3. **Then:** `parallel_workflow/llm_eassy_workflow.ipynb`
   - Combines parallel LLM calls with structured outputs
   - Demonstrates state aggregation via Annotated reducers
   - Shows how to merge and average results from concurrent evaluations

4. **Explore:** `LLM_workflow.ipynb` and `workflow_1.ipynb`
   - Additional patterns and use cases

Each notebook is self-documenting with markdown explanations and comments describing the underlying LangGraph concepts.

---

## 🎯 Workflow Examples

### Prompt-Chaining Pipeline (Sequential)

```
INPUT (blog title)
  ↓
create_outline (LLM call)
  ↓
write_content (LLM call, uses outline)
  ↓
evaluate_content (LLM call, uses content)
  ↓
OUTPUT (outline + full article + feedback)
```

**File:** `prompt_chaining_workflow.ipynb`

### Parallel Evaluation Pipeline

```
INPUT (essay)
  ↓
START
  ├→ evaluate_language (LLM)
  ├→ evaluate_clarity (LLM)
  └→ evaluate_analysis (LLM)
  ↓
final_evaluation (merge scores, aggregate, summarize)
  ↓
OUTPUT (avg_score + combined_feedback)
```

**File:** `parallel_workflow/llm_eassy_workflow.ipynb`

See `Notes/Parallel_Workflow_Revision.md` for detailed state diagrams and best practices.

---

## 📚 Key Concepts

### LangGraph Fundamentals

| Concept | Description | Reference |
|---------|-------------|-----------|
| **Graph** | Directed acyclic structure defining workflow execution order | `Langgraph_core_concepts.md` |
| **State** | Shared, typed data structure (TypedDict/Pydantic) passed through nodes | `Langgraph_core_concepts.md` |
| **Nodes** | Functions that accept state, perform work, and return state updates (deltas) | `Langgraph_core_concepts.md` |
| **Edges** | Connections between nodes; can be conditional for branching logic | `Langgraph_core_concepts.md` |

### State Management Best Practices

- **Return deltas, not full state:** Nodes should return only changed keys to prevent conflicts
- **Use TypedDict for state schema:** Provides type checking and clear documentation
- **Define merge policies:** Choose appropriate strategies for shared keys (last-writer-wins, aggregation, etc.)
- **Annotated reducers:** Use `Annotated[list, operator.add]` for explicit aggregation semantics

See `Notes/Parallel_Workflow_Revision.md` for detailed patterns and state diagrams.

---

## ✅ Best Practices

### Design Principles

- **Single Responsibility:** Each node should perform one clear task
- **Immutability:** Treat state as immutable; nodes return new deltas
- **Type Safety:** Use TypedDict or Pydantic models for all state definitions
- **Documentation:** Comment node purpose and expected state keys
- **Testing:** Validate state schema and merge logic before deployment

### Error Handling

- Wrap LLM calls in try-except blocks
- Log intermediate state for debugging
- Implement retry logic with exponential backoff for transient failures
- Validate model outputs against expected schema

### Performance Optimization

- Use parallel edges when nodes are independent
- Implement caching for repeated LLM calls (via LangChain caching)
- Monitor token usage and optimize prompts to reduce costs
- Use streaming for long-running operations when available

---

## 🔍 Troubleshooting

### "ModuleNotFoundError: No module named 'langgraph'"

```bash
# Ensure virtual environment is activated
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

# Reinstall dependencies
pip install -r requirement.txt
```

### "GROQ_API_KEY not found"

Create a `.env` file in the project root:

```
GROQ_API_KEY=your_actual_key_here
```

The notebooks use `load_dotenv()` to load this automatically.

### Jupyter notebooks not opening

```bash
# Clear Jupyter cache and try again
jupyter lab --reset
jupyter lab
```

### Import errors in notebooks

After updating dependencies, restart the kernel (Kernel → Restart in Jupyter) to reload modules.

---

## 🤝 Contributing

This is a personal learning repository, but contributions are welcome:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-pattern`)
3. Follow PEP 8 style and include type annotations
4. Add documentation in `Notes/` for new concepts
5. Test notebooks end-to-end before submitting
6. Submit a Pull Request with a clear description

---

## 📄 License

This project is licensed under the **MIT License** — see the `LICENSE` file for full terms.

---

## 🙏 Acknowledgements

- **LangGraph & LangChain** – Stateful orchestration framework and ecosystem
- **Groq** – Fast LLM inference API used in examples
- **Python community** – All open-source libraries enabling rapid AI development
- **LangChain documentation** – Official guides and API reference

---

**For more information, visit:**
- [LangGraph Official Docs](https://langchain-ai.github.io/langgraph/)
- [LangChain Documentation](https://python.langchain.com/docs)
- [Groq API Docs](https://console.groq.com/docs)

---

*Happy learning! Build scalable, production-grade AI workflows with LangGraph.* 🚀
