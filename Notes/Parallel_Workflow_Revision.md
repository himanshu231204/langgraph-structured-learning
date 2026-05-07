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
