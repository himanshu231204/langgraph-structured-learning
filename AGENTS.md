# Agent Guidelines for LangGraph Structured Learning

**Purpose:** Help AI coding agents understand repository conventions, patterns, and best practices for implementing and modifying LangGraph workflows.

> Reference [CLAUDE.md](CLAUDE.md) for commands, [README.md](README.md) for project overview, and [Notes/](Notes/) for conceptual deep-dives.

---

## Quick Navigation

| Task | Location | Pattern |
|------|----------|---------|
| **Add a new workflow** | Create `.ipynb` in root or `parallel_workflow/` | See [prompt_chaining_workflow.ipynb](prompt_chaining_workflow.ipynb) |
| **Understand parallel execution** | [parallel_workflow/simple_workflow.ipynb](parallel_workflow/simple_workflow.ipynb) | Delta merging + state convergence |
| **Learn LLM integration** | [parallel_workflow/llm_eassy_workflow.ipynb](parallel_workflow/llm_eassy_workflow.ipynb) | Structured outputs + reducers |
| **State design patterns** | [Notes/Parallel_Workflow_Revision.md](Notes/Parallel_Workflow_Revision.md) | TypedDict, merge strategies, state diagrams |
| **Core concepts** | [Notes/Langgraph_core_concepts.md](Notes/Langgraph_core_concepts.md) | Graph, State, Nodes, Edges theory |

---

## Common Agent Tasks & Patterns

### 1. **Create a New Notebook from Scratch**

**Use as template:** [prompt_chaining_workflow.ipynb](prompt_chaining_workflow.ipynb)

**Boilerplate to always include:**

```python
# Cell 1: Imports
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Annotated
from langchain_groq import ChatGroq
from dotenv import load_dotenv
import operator

# Cell 2: Load environment
load_dotenv()

# Cell 3: Initialize LLM
model = ChatGroq(model="llama-3.3-70b-versatile")

# Cell 4: Define state schema
class WorkflowState(TypedDict):
    input_field: str
    output_field: str
    # optional: aggregation_field: Annotated[list, operator.add]

# Cell 5+: Define node functions (each returns partial dict)
def node_a(state: WorkflowState) -> dict:
    result = model.invoke(f"Process: {state['input_field']}")
    return {"output_field": result}

# Cell N: Build graph
graph = StateGraph(WorkflowState)
graph.add_node("node_a", node_a)
graph.add_edge(START, "node_a")
graph.add_edge("node_a", END)
workflow = graph.compile()

# Cell N+1: Execute
initial_state = {"input_field": "test"}
result = workflow.invoke(initial_state)
```

**Critical rules:**
- ✅ State schema is a `TypedDict` with clear field names
- ✅ Each node returns a **delta** (partial dict), not full state
- ✅ Call `load_dotenv()` before creating model
- ✅ Always include state diagram comment (see below)

---

### 2. **Add State Diagrams (Required)**

**User preference:** Always include state diagrams for clarity.

**When adding/modifying workflows, include diagram:**

```python
"""
State Flow Diagram:

START
  |
  +---> node_a (process input) ──┐
  |                               |
  +---> node_b (compute metric) ──┤ (merge deltas)
  |                               |
  +---> node_c (validate) ────────┘
                                |
                        node_d (aggregate)
                                |
                               END
"""
```

**Tools used:** ASCII diagrams in code comments or markdown cells. For complex workflows, add diagram to corresponding `.md` file in `Notes/`.

---

### 3. **Parallel Workflows: The State Merging Pattern**

**Reference:** [parallel_workflow/simple_workflow.ipynb](parallel_workflow/simple_workflow.ipynb)

**Key principle:** Return deltas; LangGraph merges them automatically.

```python
# ❌ WRONG: Return full state (overwrites other nodes' work)
def node_a(state):
    return {"field_a": 100, "field_b": 0, "field_c": 0}

# ✅ CORRECT: Return only what changed
def node_a(state):
    return {"field_a": 100}  # Other nodes add their fields
```

**Merge behavior:**
```python
# After parallel nodes a, b, c complete, state becomes:
merged = {
    "field_a": 100,      # from node_a
    "field_b": 200,      # from node_b
    "field_c": 300,      # from node_c
    "original_key": "...", # preserved from initial state
}
```

---

### 4. **Structured LLM Outputs with Pydantic**

**Reference:** [parallel_workflow/llm_eassy_workflow.ipynb](parallel_workflow/llm_eassy_workflow.ipynb)

**Pattern for type-safe LLM responses:**

```python
from pydantic import BaseModel, Field

class ResponseSchema(BaseModel):
    score: int = Field(..., ge=0, le=10, description="Score 0-10")
    feedback: str = Field(..., description="Detailed feedback")

# Bind schema to model
structured_model = model.with_structured_output(ResponseSchema)

# Use it
response = structured_model.invoke(prompt)  # Returns ResponseSchema instance
print(response.score, response.feedback)
```

**In nodes:** Return structured output fields in the delta dict.

---

### 5. **State Aggregation with Annotated Reducers**

**Reference:** [parallel_workflow/llm_eassy_workflow.ipynb](parallel_workflow/llm_eassy_workflow.ipynb)

**Pattern for combining results from parallel nodes:**

```python
from typing import Annotated
import operator

class EvaluationState(TypedDict):
    results: Annotated[list, operator.add]  # Scores will be added together

def parallel_node_1(state):
    return {"results": [7]}  # Returns list with one score

def parallel_node_2(state):
    return {"results": [8]}  # Returns list with one score

# After merge: results = [7, 8]
# Then in final node: avg_score = sum(results) / len(results)
```

**Reducers:**
- `operator.add` – lists concatenate: `[7] + [8] = [7, 8]`
- `operator.mul` – multiply (rarely used)
- Custom – write `lambda a, b: a + b` for sum, `max(a, b)` for max, etc.

---

### 6. **Conditional Edges for Branching**

**Pattern for runtime branching based on state:**

```python
def should_continue(state: WorkflowState) -> str:
    if state["score"] > 7:
        return "high_quality_path"
    else:
        return "retry_path"

graph.add_conditional_edges(
    "evaluate",
    should_continue,
    {
        "high_quality_path": "publish",
        "retry_path": "revise"
    }
)
```

---

## Expected Notebook Structure

Every notebook should follow this cell sequence:

| Cell(s) | Content | Purpose |
|---------|---------|---------|
| 1-3 | Imports, `load_dotenv()`, model init | Setup |
| 4-6 | State schema, Pydantic models, constants | Definition |
| 7-10 | Node function definitions | Computation logic |
| 11-15 | Graph building (`StateGraph`, `add_node`, `add_edge`, `compile`) | Graph assembly |
| 16+ | Execution, result inspection, debugging | Testing & validation |

**Documentation:** Each major cell block should have a markdown cell explaining its purpose.

---

## Debugging Checklist

When a notebook fails, check:

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: langgraph` | Activate venv; restart Jupyter kernel |
| `KeyError` in state access | Validate `initial_state` has all required TypedDict keys |
| Node returns incorrect type | Ensure node returns a `dict`, not full state object |
| Parallel nodes overwrite each other | Make sure each node returns delta (only its keys), not full state |
| `GROQ_API_KEY` not found | Create `.env` file; call `load_dotenv()` before model init |
| LLM response not structured | Use `model.with_structured_output(Schema)` |
| State not merged correctly | Check Annotated reducer syntax: `Annotated[list, operator.add]` |

---

## Code Style & Conventions

### Naming

- **State classes:** `PascalCase` + `State` suffix (e.g., `BlogState`, `EssayState`)
- **Node functions:** `snake_case`, descriptive (e.g., `create_outline`, `evaluate_language`)
- **Graph nodes:** Match function name or `snake_case_path` (e.g., `"generate_summary"`)
- **Edge labels:** Same as node names or descriptive (e.g., `"high_quality"`, `"retry"`)

### Type Hints

- Always annotate function parameters: `def node_fn(state: BlogState) -> dict:`
- Use TypedDict for state: `class BlogState(TypedDict): field: str`
- Import from `typing`: `TypedDict, Annotated`, and `operator` for reducers

### Comments

- Add diagram comment in graph-building cell
- Document each node's purpose and expected input/output keys
- Mark Annotated reducers with what operation they perform

---

## Common Modifications AI Agents Make

### Adding a New Node

1. Define function: `def new_node(state: WorkflowState) -> dict:`
2. Return delta: `return {"new_key": value}`
3. Register: `graph.add_node("new_node", new_node)`
4. Connect edges: `graph.add_edge("prev_node", "new_node")` and `graph.add_edge("new_node", "next_node")`
5. Update diagram comment in graph-building cell
6. Test with `workflow.invoke(initial_state)`

### Modifying Node Logic

- Edit node function in its cell
- Re-run node function cell
- Re-run graph compilation cell (must recompile after function changes)
- Re-run execution cell to test

### Changing State Schema

- Edit `TypedDict` definition
- Add/remove fields
- Update `initial_state` dict in execution cell to match schema
- Update all node functions to return correct delta keys
- Recompile graph
- Test

### Adding Parallel Execution

- Identify independent nodes
- Add multiple edges from `START` to each node
- Add edges from each node to next convergence point
- Ensure each node returns only its own keys (deltas)
- Include updated state diagram

---

## File Organization Guidelines

- **Root `.ipynb` files:** Sequential workflows (e.g., `prompt_chaining_workflow.ipynb`)
- **`parallel_workflow/` folder:** Parallel execution examples
- **`Notes/` folder:** Conceptual documentation & best practices
  - `Langgraph_core_concepts.md` – Theory
  - `Parallel_Workflow_Revision.md` – Patterns with state diagrams
  - New notes go here for reusable knowledge

---

## References & Links

- [CLAUDE.md](CLAUDE.md) – Commands and high-level architecture
- [README.md](README.md) – Project overview, quick start, troubleshooting
- [ARCHITECTURE.md](ARCHITECTURE.md) – Detailed component relationships, design decisions, data flow patterns
- [Notes/Langgraph_core_concepts.md](Notes/Langgraph_core_concepts.md) – LangGraph theory
- [Notes/Parallel_Workflow_Revision.md](Notes/Parallel_Workflow_Revision.md) – Parallel patterns with diagrams
- [LangGraph Official Docs](https://langchain-ai.github.io/langgraph/)

---

## Quick Checklist Before Committing Changes

- ✅ State schema defined (TypedDict)
- ✅ All nodes return deltas (partial dicts)
- ✅ Graph diagram included in code comment
- ✅ Notebook executes end-to-end without errors
- ✅ Type hints on all functions
- ✅ Markdown cells explain key sections
- ✅ `.env` file not committed (in `.gitignore`)
- ✅ Updated `Notes/` documentation if creating reusable patterns

---

*End of AGENTS.md*
