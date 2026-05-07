# Parallel Workflow — Revision (LangGraph)

## Overview

LangGraph workflows use a `StateGraph` that holds a shared typed `state` (TypedDict or Pydantic model). Nodes are small functions that perform one responsibility and produce outputs that are merged back into the global state. Parallel workflows run multiple nodes concurrently and then merge their outputs.

## Why return partial dicts (deltas) instead of the full state

- Minimize conflicts: parallel nodes often update different keys. Returning only changed keys prevents accidental overwrites of unrelated fields.
- Predictable merging: shallow dict merges (state.update(node_output)) are simple and deterministic.
- Smaller payloads: sending deltas is cheaper to validate and serialize.
- Clear intent: node outputs explicitly show which fields changed, easing validation and debugging.
- Easier conflict resolution: when two nodes touch the same key, explicit merge policies (last-writer-wins, aggregator, or error) can be applied.

## Typical merge strategies

- Shallow merge (default): for each node_output do `state.update(node_output)`.
- Aggregation: for shared keys, use a reducer function (sum, average, choose-max).
- Locking/serialization: serialize updates to a shared key (less common; use for critical sections).
- Schema validation: validate merged state against the TypedDict/Pydantic model.

## Example pattern

```
START
  |
  +---> node_a (compute strike_rate: 42) ──┐
  |                                         | (merge deltas)
  +---> node_b (compute boundaries: 3) ────┘
                                      |
                            downstream_node
                                      |
                                     END
```

Code example:
```python
def node_a(state):
    # compute only what changed
    return {"strike_rate": 42}

def node_b(state):
    return {"boundaries": 3}

# run in parallel -> outputs = [out_a, out_b]
# merge:
for out in outputs:
    state.update(out)

# validate state against schema
```

## Best practices

- Keep nodes single-responsibility and return only their outputs (a dict of changed keys).
- Define a clear state schema so merges can be validated.
- Avoid in-place full-state returns in parallel branches.
- Document keys each node may produce.
- Decide and document conflict resolution rules for shared keys.

## Quick checklist

- [ ] Node returns only changed keys
- [ ] State schema defined and validated
- [ ] Merge policy for conflicts chosen
- [ ] Nodes documented with output keys

---
_Created to revise and remember the LangGraph parallel-workflow pattern._

## Notebook-specific concepts

- `parallel_workflow/simple_workflow.ipynb` — Non-LLM example:
    - **State type:** `BatsmanState` (a `TypedDict`).
    - **Nodes:** `calculate_strike_rate`, `calculate_boundries_per_balls`, `calculate_boundries_percent`, `generate_summary`.
    - **Pattern:** `START` -> three metric nodes run in parallel -> `generate_summary` -> `END`.
    - **Outputs:** nodes return partial dicts (deltas) such as `{'strike_rate': ...}` that are merged into the global state.
    - **Purpose:** demonstrates pure-Python parallel computation, safe merging of deltas, and a final summarization node dependent on prior outputs.
  - **State diagram:**
    ```
    START
      |
      +---> calculate_strike_rate ──┐
      |                              |
      +---> calculate_boundries_per_balls ──┤ (merge deltas)
      |                              |
      +---> calculate_boundries_percent ──┘
                                    |
                            generate_summary
                                    |
                                   END
    ```
  - **Example snippet (Cell 7–9):**
    ```python
    def calculate_strike_rate(state: BatsmanState) -> BatsmanState:
        strike_rate = (state['runs'] / state['balls']) * 100
        return {'strike_rate': strike_rate}  # delta only
    
    # Three nodes from START run in parallel, each returning a delta
    graph.add_edge(START, "calculate_strike_rate")
    graph.add_edge(START, "calculate_boundries_per_balls")
    graph.add_edge(START, "calculate_boundries_percent")
    # All merge into one state, then feed to generate_summary
    ```

- `parallel_workflow/llm_eassy_workflow.ipynb` — LLM-backed evaluation pipeline:
    - **Model & schema:** uses `ChatGroq` with `with_structured_output(EvaluationSchema)` (a `pydantic.BaseModel`) for typed LLM responses.
    - **State type:** `EssayState` (`TypedDict`) with `individual_score: Annotated[list[int], operator.add]` to indicate aggregation behavior.
    - **Nodes:** `evulate_language`, `evulate_clarity`, `evulate_analysis` each call the structured LLM and return `{'..._feedback': ..., 'individual_score': [score]}`.
    - **Aggregation:** parallel LLM evaluations produce lists of scores which are merged (via the annotated reducer) and then averaged in `final_evaluation`; `final_evaluation` also calls the LLM to produce an overall summary.
    - **Purpose:** shows how to combine structured LLM outputs, typed schemas, and reducer-based aggregation in a parallel workflow.
  - **State diagram:**
    ```
    START (essay in state)
      |
      +---> evulate_language ──────┐
      |     (call LLM, score: 8)    |
      +---> evulate_clarity ────────┤ (merge: individual_score = [8, 7, 8])
      |     (call LLM, score: 7)    |
      +---> evulate_analysis ───────┘
            (call LLM, score: 8)
                      |
            final_evaluation
            (avg_score = 7.67, call LLM for summary)
                      |
                     END
    ```
  - **Example snippet (Cell 10–13, 17):**
    ```python
    from pydantic import BaseModel, Field
    from typing import Annotated
    import operator
    
    class EvaluationSchema(BaseModel):
        score: int = Field(..., ge=0, le=10)
        feedback: str = Field(...)
    
    class EssayState(TypedDict):
        individual_score: Annotated[list[int], operator.add]  # reducer: list addition
    
    def evulate_language(state: EssayState):
        output = structured_model.invoke(prompt)
        return {'individual_score': [output.score], 'language_feedback': output.feedback}
    
    # evulate_clarity, evulate_analysis run in parallel, each appending a score
    # final_evaluation merges scores and calls LLM for overall summary
    ```

- **Shared concepts across both notebooks:**
    - START-based parallel branching with convergence to a final node.
    - Nodes return deltas (partial dicts) to avoid overwriting unrelated keys.
    - Use of a typed state (TypedDict / Pydantic) to document and validate keys.
    - Final node performs aggregation/summary after parallel branches complete.
    - Optional: annotated reducers (like `operator.add`) for explicit aggregation semantics.

---
