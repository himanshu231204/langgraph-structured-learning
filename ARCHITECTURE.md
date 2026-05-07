# ARCHITECTURE.md

## Project Architecture: LangGraph Structured Learning

This document explains the overall architecture, component relationships, and design decisions in the LangGraph Structured Learning repository.

---

## Table of Contents

- [Overview](#overview)
- [Core Concepts](#core-concepts)
- [Notebook Architecture](#notebook-architecture)
- [State Management Strategy](#state-management-strategy)
- [Component Relationships](#component-relationships)
- [Learning Progression Model](#learning-progression-model)
- [Design Decisions](#design-decisions)
- [Data Flow Patterns](#data-flow-patterns)
- [Extension Points](#extension-points)

---

## Overview

**Purpose:** This repository is a **structured learning resource** designed to teach LangGraph—a stateful workflow orchestration framework—through progressive, hands-on examples.

**Core Philosophy:**
- **Learning by example:** Real, runnable notebooks demonstrate concepts progressively
- **State as first-class:** All workflows emphasize shared, typed state passing through graphs
- **Theory + Practice:** Conceptual notes paired with executable notebooks for immediate verification
- **Pattern reuse:** Notebooks serve as templates for production workflows

**Target Audience:**
- Developers learning LangGraph for the first time
- Teams building AI agents or RAG systems
- Engineers migrating from stateless chains to stateful workflows

---

## Core Concepts

### LangGraph Fundamentals

LangGraph provides four essential building blocks:

| Concept | Role | Example |
|---------|------|---------|
| **Graph** | DAG (directed acyclic graph) defining execution order | `StateGraph(State)` → add nodes/edges → `compile()` |
| **State** | Typed, shared data structure passed between nodes | `class BlogState(TypedDict): outline: str, content: str` |
| **Node** | Function performing one responsibility, returns state delta | `def write_content(state) -> dict: return {"content": ...}` |
| **Edge** | Connection between nodes; can be conditional | `START → node_a → node_b → END` or conditional routing |

### Key Architectural Patterns

This repository demonstrates three primary patterns:

1. **Sequential Chaining** – One node after another; state passed linearly
   - Use case: Multi-step LLM workflows (outline → write → evaluate)
   - Example: `prompt_chaining_workflow.ipynb`

2. **Parallel Execution** – Multiple nodes run concurrently; deltas merge
   - Use case: Independent evaluations, concurrent tool calls
   - Example: `parallel_workflow/simple_workflow.ipynb`, `parallel_workflow/llm_eassy_workflow.ipynb`

3. **Conditional Routing** – Branches based on state values
   - Use case: Approval gates, retry logic, branching pipelines
   - Example: Demonstrated in AGENTS.md Section 6

---

## Notebook Architecture

### File Organization

```
langgraph-structured-learning/
├── 📓 Root Notebooks (Sequential Workflows)
│   ├── prompt_chaining_workflow.ipynb      ⭐ PRIMARY TEMPLATE
│   ├── LLM_workflow.ipynb
│   └── workflow_1.ipynb
│
├── 📓 parallel_workflow/ (Parallel Execution Patterns)
│   ├── simple_workflow.ipynb               ⭐ FOUNDATION (non-LLM)
│   └── llm_eassy_workflow.ipynb            ⭐ ADVANCED (LLM + aggregation)
│
├── 📚 Notes/ (Conceptual Documentation)
│   ├── Langgraph_core_concepts.md          (Theory)
│   ├── Parallel_Workflow_Revision.md       (Patterns with diagrams)
│   └── LangGraph_Four_Core_Concepts_Deep_Dive_for_Beginners.md
│
├── 📄 Configuration & Guidance
│   ├── AGENTS.md                           (AI agent instructions)
│   ├── CLAUDE.md                           (Command reference)
│   ├── ARCHITECTURE.md                     (This file)
│   ├── README.md                           (Quick start & features)
│   └── requirement.txt                     (Dependencies)
```

### Notebook Roles & Dependencies

#### **prompt_chaining_workflow.ipynb** (⭐ Primary Template)

**Role:** Canonical starting point for learning LangGraph

**Key Patterns Taught:**
- State schema definition with `TypedDict`
- Sequential node arrangement (`START → node_a → node_b → ... → END`)
- Node function structure (input state, output delta)
- Graph building and compilation
- State inspection after execution

**State Flow:**
```
initial_state: {"title": "..."}
    ↓
create_outline(state) → {"outline": "..."}
    ↓
write_content(state) → {"content": "..."}
    ↓
evaluate_content(state) → {"feedback": "..."}
    ↓
final_state: {"title", "outline", "content", "feedback"}
```

**Should Copy From This When:**
- Creating a new sequential LLM workflow
- Demonstrating prompt chaining
- Teaching basic LangGraph concepts

---

#### **parallel_workflow/simple_workflow.ipynb** (⭐ Foundation: Non-LLM)

**Role:** Teach parallel execution without LLM complexity

**Key Patterns Taught:**
- Parallel node execution (multiple `START` edges)
- Delta merging (nodes return only their keys)
- Convergence point (all parallel branches feed to one node)
- State preservation across merges

**State Flow:**
```
initial_state: {"runs": 120, "balls": 80, "fours": 10, "sixes": 5}
    ↓
START (fan-out to 3 parallel nodes)
├─ calculate_strike_rate(state) → {"strike_rate": 150.0}
├─ calculate_boundries_per_balls(state) → {"boundries_per_balls": 0.3}
└─ calculate_boundries_percent(state) → {"boundries_percent": 30.0}
    ↓
merged_state: {all above keys + originals}
    ↓
generate_summary(state) → {"summary": "..."}
    ↓
final_state: {all keys combined}
```

**Design Goal:** Pure computation example removes LLM/API complexity; learners focus on:
- How LangGraph merges parallel outputs
- Why returning deltas (not full state) prevents conflicts
- How convergence nodes can use parallel results

**Should Copy From This When:**
- Learning parallel execution without LLM calls
- Debugging state merging issues
- Understanding delta concept

---

#### **parallel_workflow/llm_eassy_workflow.ipynb** (⭐ Advanced: Parallel + LLM + Aggregation)

**Role:** Demonstrate production-pattern: parallel LLM calls with structured outputs and aggregation

**Key Patterns Taught:**
- Parallel LLM node execution (3 concurrent evaluators)
- Structured LLM outputs with Pydantic schemas (`BaseModel`)
- Annotated reducers (`Annotated[list[int], operator.add]`)
- State aggregation (merging lists from parallel nodes)
- Final aggregation node (computing average from list)

**State Flow:**
```
initial_state: {"essay": "..."}
    ↓
START (fan-out to 3 parallel LLM calls)
├─ evaluate_language(state) 
│  → {"language_feedback": "...", "individual_score": [8]}
├─ evaluate_clarity(state)
│  → {"clarity_feedback": "...", "individual_score": [7]}
└─ evaluate_analysis(state)
   → {"analysis_feedback": "...", "individual_score": [9]}
    ↓
merged_state via Annotated reducer:
  {"individual_score": [8, 7, 9], language_feedback, clarity_feedback, analysis_feedback}
    ↓
final_evaluation(state)
  → avg_score = sum([8,7,9])/3 = 8.0
  → {"avg_score": 8.0, "overall_feedback": "..."}
    ↓
final_state: {all keys + computed avg}
```

**Design Goal:** Production-grade pattern combining:
- Parallel independent work streams (3 evaluators)
- Structured outputs (type-safe LLM responses)
- Aggregation semantics (explicit reducer function)

**Should Copy From This When:**
- Building multi-evaluator systems
- Using structured LLM outputs
- Aggregating results from parallel LLM nodes

---

#### **LLM_workflow.ipynb, workflow_1.ipynb** (Support Examples)

**Role:** Additional pattern demonstrations; alternative or complementary approaches

**Usage:** Reference for specific use cases not covered in primary templates

---

### Notes Structure

#### **Notes/Langgraph_core_concepts.md**

Conceptual foundation explaining:
- Graph as DAG
- State as immutable record
- Nodes as pure functions
- Edges as directed connections

**Link to:** Read before starting notebooks for theory; reference while implementing

---

#### **Notes/Parallel_Workflow_Revision.md**

Advanced state management patterns:
- Why return deltas instead of full state
- Merge strategies (shallow, aggregation, custom)
- State diagrams for both parallel examples
- Best practices checklist

**Link to:** Reference when implementing parallel workflows; copy state diagrams into new notebooks

---

## State Management Strategy

### State Schema Design

**Rule 1: Always use `TypedDict`**

```python
class WorkflowState(TypedDict):
    # Input fields (immutable during execution)
    input_text: str
    
    # Processing fields (modified by nodes)
    processed_output: str
    
    # Aggregation fields (combine from parallel)
    results: Annotated[list, operator.add]
    
    # Final computed fields
    summary: str
```

**Benefits:**
- Type checking (IDE autocomplete)
- Self-documenting (field purposes clear)
- Schema validation
- Clear state contract for all nodes

### Delta Pattern (Return Only Changed Keys)

**❌ Anti-pattern:**
```python
def node_a(state: State) -> State:
    # Returns full state; will overwrite other nodes' work in parallel
    return {
        "field_a": 100,
        "field_b": 0,  # Overwrites field_b from node_b!
        "field_c": 0   # Overwrites field_c from node_c!
    }
```

**✅ Correct pattern:**
```python
def node_a(state: State) -> dict:
    # Returns only what this node computed
    return {"field_a": 100}
    # Other fields untouched; safe in parallel
```

### Merge Semantics

**Default (Shallow Merge):**
```python
# After parallel nodes finish:
final_state = {
    **initial_state,           # Original fields preserved
    **node_a_output,           # {"field_a": 100}
    **node_b_output,           # {"field_b": 200}
    **node_c_output            # {"field_c": 300}
}
# Result: all fields present, no conflicts
```

**Aggregation (with Annotated Reducer):**
```python
class State(TypedDict):
    scores: Annotated[list[int], operator.add]

def node_1(state) -> dict:
    return {"scores": [7]}

def node_2(state) -> dict:
    return {"scores": [8]}

# After merge: scores = [7] + [8] = [7, 8]
```

---

## Component Relationships

### Dependency Graph

```
Langgraph Core Concepts (Theory)
    ↓
    ├─→ prompt_chaining_workflow.ipynb (Sequential)
    │       ↓
    │   Teaches: Graph, State, Nodes, Edges
    │
    └─→ parallel_workflow/simple_workflow.ipynb (Parallel, no LLM)
            ↓
        Teaches: Delta merging, convergence
            ↓
        parallel_workflow/llm_eassy_workflow.ipynb (Parallel + LLM + Aggregation)
                ↓
            Teaches: Structured outputs, reducers, aggregation
```

### Cross-References

| File | References | Purpose |
|------|-----------|---------|
| AGENTS.md | All notebooks | Tells AI agents which pattern to copy |
| CLAUDE.md | AGENTS.md | Provides commands, architecture overview |
| README.md | All files | Provides quick start and troubleshooting |
| Parallel_Workflow_Revision.md | simple_workflow.ipynb, llm_eassy_workflow.ipynb | Explains patterns with diagrams |

---

## Learning Progression Model

### Recommended Order for Learning

#### **Level 1: Fundamentals (Beginners)**

1. Read: `Notes/Langgraph_core_concepts.md`
2. Run: `prompt_chaining_workflow.ipynb`
3. Modify: Add a node to the chain, recompile, test

**Learning outcomes:**
- Understand StateGraph, START, END
- See state pass through sequential nodes
- Write a simple node function
- Compile and invoke a workflow

#### **Level 2: Concurrency (Intermediate)**

1. Read: `Notes/Parallel_Workflow_Revision.md`
2. Run: `parallel_workflow/simple_workflow.ipynb`
3. Experiment: Change a node to return full state instead of delta; observe merge conflict

**Learning outcomes:**
- Understand parallel execution
- Why delta (partial dict) is safer than full state
- How LangGraph merges concurrent outputs
- Convergence points and dependency resolution

#### **Level 3: Production Patterns (Advanced)**

1. Run: `parallel_workflow/llm_eassy_workflow.ipynb`
2. Study: Pydantic schema, Annotated reducers, final aggregation
3. Build: Create a new parallel workflow with 2 independent LLM evaluators

**Learning outcomes:**
- Structured LLM outputs (type safety)
- Aggregation semantics (reducer functions)
- Combining parallel streams
- Production-ready error handling

#### **Level 4: Custom Extensions**

1. Reference: `AGENTS.md` for task patterns
2. Build: New workflow combining learned patterns
3. Document: Add state diagram and notes to `Notes/`

---

## Design Decisions

### Why TypedDict Instead of Pydantic for State?

**Decision:** State schemas use `TypedDict` (not Pydantic `BaseModel`)

**Rationale:**
- TypedDict is simpler and faster for state (no validation overhead per node)
- Pydantic used for LLM outputs (structured, validated responses) where type safety is critical
- Separation of concerns: TypedDict for graph state, Pydantic for external APIs

**Trade-off:** Less validation at state boundaries (but acceptable for learning repo)

---

### Why Parallel Examples Are Separate Notebooks

**Decision:** `parallel_workflow/` is separate directory with dedicated notebooks

**Rationale:**
- Sequential pattern is simplest, learned first
- Parallel adds complexity; separate notebook avoids confusing beginners
- Allows progression: non-LLM parallel → LLM parallel
- Clear file organization by pattern type

---

### Why State Diagrams Are ASCII, Not Mermaid

**Decision:** ASCII diagrams in markdown code blocks

**Rationale:**
- User preference: ASCII diagrams show clearly in comments and notes
- Portable: Works in notebooks, markdown, plain text
- Simple to edit: No special tooling needed
- Future upgrade: Can generate Mermaid on demand if needed

---

### Why Groq LLM (Not OpenAI, Claude, etc.)

**Decision:** `ChatGroq` for LLM integration

**Rationale:**
- Fast inference (low latency)
- Affordable for learning/examples
- LLM choice is orthogonal to learning LangGraph
- Easy to swap out: Replace `ChatGroq` with `ChatOpenAI` if needed

---

## Data Flow Patterns

### Sequential Pattern

```
INPUT (title)
  ↓
create_outline (reads: title, writes: outline)
  ↓ [state now has: title, outline]
write_content (reads: title + outline, writes: content)
  ↓ [state now has: title, outline, content]
evaluate_content (reads: content, writes: feedback)
  ↓
OUTPUT (title, outline, content, feedback)
```

**Data characteristic:** Accumulative; each node adds to state, nothing removed

---

### Parallel Pattern

```
INPUT (runs, balls, fours, sixes)
  ↓ (fan-out to 3 concurrent processes)
  ├─ calculate_strike_rate
  │  (reads: runs, balls | writes: strike_rate)
  ├─ calculate_boundries_per_balls
  │  (reads: fours, sixes, balls | writes: boundries_per_balls)
  └─ calculate_boundries_percent
     (reads: fours, sixes, runs | writes: boundries_percent)
  ↓ (merge: LangGraph combines all outputs)
  │ merged_state = {original + strike_rate + boundries_per_balls + boundries_percent}
  ↓ (convergence)
generate_summary (reads: all computed fields, writes: summary)
  ↓
OUTPUT (original + all computed fields + summary)
```

**Data characteristic:** Parallel reads, partial writes, merge-and-converge

---

### Aggregation Pattern (Advanced)

```
INPUT (essay)
  ↓ (fan-out to 3 parallel LLM evaluators)
  ├─ evaluate_language
  │  (LLM call | writes: language_feedback, individual_score: [8])
  ├─ evaluate_clarity
  │  (LLM call | writes: clarity_feedback, individual_score: [7])
  └─ evaluate_analysis
     (LLM call | writes: analysis_feedback, individual_score: [9])
  ↓ (merge with Annotated reducer)
  │ individual_score lists aggregate: [8] + [7] + [9] = [8, 7, 9]
  ↓ (convergence)
final_evaluation
  (reads: all feedback fields + aggregated scores | writes: avg_score, overall_feedback)
  │ avg_score = sum([8, 7, 9]) / 3 = 8.0
  ↓
OUTPUT (all feedback + avg_score + overall_feedback)
```

**Data characteristic:** Parallel LLM calls, list aggregation via reducer, computed final metric

---

## Extension Points

### Where to Add New Workflows

1. **Sequential workflow?** → Root `.ipynb` file (copy `prompt_chaining_workflow.ipynb`)
2. **Non-LLM parallel?** → `parallel_workflow/` (copy `simple_workflow.ipynb`)
3. **Parallel + LLM?** → `parallel_workflow/` (copy `llm_eassy_workflow.ipynb`)

### Where to Add New Concepts

1. **Theoretical explanation?** → `Notes/` new `.md` file
2. **Pattern with diagram?** → Add state diagram to `Notes/Parallel_Workflow_Revision.md` or new file
3. **Agent task pattern?** → Update `AGENTS.md` Section "Common Agent Tasks & Patterns"

### Where to Add New Documentation

1. **Quick reference?** → Update `CLAUDE.md`
2. **Feature overview?** → Update `README.md` Features table
3. **Troubleshooting?** → Update `README.md` Troubleshooting section
4. **Agent guidance?** → Update `AGENTS.md`

---

## Architectural Constraints

### What We DO

✅ Teach LangGraph patterns through runnable notebooks  
✅ Progressive complexity: sequential → parallel → aggregation  
✅ Type-safe state with TypedDict  
✅ Production patterns (structured outputs, error handling)  
✅ Link to external documentation (don't duplicate)  

### What We DON'T

❌ Build production AI agents (framework only)  
❌ Add complex multi-step pipelines (keep examples focused)  
❌ Test suite (future; repo is educational)  
❌ Database/logging infrastructure (keep lightweight)  

---

## Summary

**This repository demonstrates three core patterns:**

| Pattern | Example | Learning Level |
|---------|---------|-----------------|
| **Sequential** | Blog generation (outline → content → feedback) | Beginner |
| **Parallel (Computation)** | Cricket stats (concurrent calculations) | Intermediate |
| **Parallel (LLM + Aggregation)** | Essay evaluation (parallel LLM, combined scores) | Advanced |

**Each pattern builds on previous learning**, using TypedDict for state, deltas for updates, and explicit diagrams for clarity.

**For agents and humans alike:** Follow `AGENTS.md` for task patterns, `Notes/` for concepts, and notebooks for runnable examples.

---

*End of ARCHITECTURE.md*
