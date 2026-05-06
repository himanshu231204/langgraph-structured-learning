# Revision Notes — LangGraph Workflows

## Purpose
Concise summary of concepts and implementations used across the three notebooks: `LLM_workflow.ipynb`, `prompt_chaining_workflow.ipynb`, and `workflow_1.ipynb`.

## Common Concepts
- **StateGraph**: core workflow abstraction from `langgraph.graph`. Build a directed graph of nodes operating on a shared state.
- **TypedDict states**: each workflow defines a `TypedDict` (e.g., `llmstate`, `BlogState`, `BMIState`) to declare the schema of the mutable state passed between nodes.
- **START / END**: sentinel node names used to mark graph entry and exit.
- **Nodes**: plain Python functions that accept the state, perform one responsibility, update or return the state, and return it. Registered with `graph.add_node(name, fn)`.
- **Edges**: connections between nodes declared with `graph.add_edge(src, dst)`. They define execution order.
- **Compile & Invoke**: call `graph.compile()` to build a `workflow`, then `workflow.invoke(initial_state)` to execute it.
- **LLM integration**: notebooks use `ChatGroq` from `langchain_groq` (e.g., `model = ChatGroq(...)`) and call `model.invoke(prompt)` (often using `.content`) to get responses.
- **Prompts & Patterns**: nodes construct prompt strings, use the model to generate: outlines, content, evaluations, or direct answers.
- **Visualization**: use `workflow.get_graph().draw_mermaid_png()` to render a Mermaid diagram of the graph.
- **Environment**: `dotenv.load_dotenv()` is used to load API keys before instantiating model clients.

## Notebook-specific Notes

- `LLM_workflow.ipynb`:
  - State: `llmstate` with `question` and `answer`.
  - Node: `llm_qa(state)` — builds a prompt `Answer the following question: {question}`, calls `model.invoke(prompt)`, updates `state['answer']` and returns the state.
  - Graph: single-node graph connecting `START -> llm_qa -> END` and invoking with `{'question': ...}`.

- `prompt_chaining_workflow.ipynb`:
  - State: `BlogState` with `title`, `outline`, `content`, `feedback`.
  - Nodes:
    - `create_outline`: prompts model to generate an outline from `title`, sets `state['outline']`.
    - `write_content`: prompts model to write the article based on `title` and `outline`, sets `state['content']`.
    - `evaluate_content`: prompts model to rank and provide feedback, sets `state['feedback']`.
  - Graph: linear chain `START -> create_outline -> write_content -> evaluate_content -> END`.
  - Pattern: prompt-chaining—each node enriches the state and passes it forward.

- `workflow_1.ipynb` (BMI example):
  - State: `BMIState` with `weight_kg`, `height_m`, `bmi`.
  - Node: `calculate_bmi` computes `bmi = weight/(height**2)` and returns the updated state (demonstrates non-LLM numeric computation).
  - Purpose: shows a simple pure-function node and the same graph/compile/invoke pattern used for non-LLM tasks.

## Implementation Tips & Best Practices
- Keep nodes focused: each node should perform a single, testable responsibility.
- State consistency: ensure nodes return the expected `TypedDict` shape. Either mutate the incoming dict and return it, or return a new dict consistent with the TypedDict.
- Use `.content` when the LLM client returns response objects with content attributes (not all client APIs are identical).
- Always call `load_dotenv()` before creating model clients if you rely on environment variables.
- Validate `initial_state` contains required keys before invoking the workflow to avoid runtime KeyError.
- Use Mermaid visualization to verify graph structure during development.

## Quick Code Patterns

- Minimal node registration and execution:

  1. Define state TypedDict.
  2. Implement node functions: `def my_node(state: MyState) -> MyState:`
  3. Build graph: `graph = StateGraph(MyState)`
  4. Add nodes and edges: `graph.add_node('my_node', my_node)` / `graph.add_edge(START,'my_node')`
  5. Compile and run: `workflow = graph.compile()` then `workflow.invoke(initial_state)`

## Where to Look
- Examples in repository:
  - LLM example: [LLM_workflow.ipynb](LLM_workflow.ipynb)
  - Prompt-chaining example: [prompt_chaining_workflow.ipynb](prompt_chaining_workflow.ipynb)
  - Numeric workflow example: [workflow_1.ipynb](workflow_1.ipynb)

---
Generated from the three notebooks to help revision and reuse.
