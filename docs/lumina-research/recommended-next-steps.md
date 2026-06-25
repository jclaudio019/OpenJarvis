# Recommended Next Steps

## Strategic Recommendation: Partial Extraction and Adapter Pattern

Lumina should **not** use OpenJarvis directly as a core library, nor should it rebuild everything internally. Instead, Lumina should adopt a hybrid approach: **Partially Extract** key data structures and **Wrap** execution engines through adapters.

### 1. Extract Intelligence Configurations (Priority: High)
- **Action**: Copy and adapt `src/openjarvis/intelligence/model_catalog.py` and telemetry data classes from `src/openjarvis/core/types.py`.
- **Value**: Provides immediate access to a comprehensive, well-researched database of model capabilities, contexts, and costs without importing dependencies.
- **Complexity**: Low. Pure data structures.

### 2. Wrap Inference Engines (Priority: High)
- **Action**: Build an Adapter (as outlined in `adapter-recommendation.md`) that targets `src/openjarvis/engine/`.
- **Value**: Delegates the complex maintenance of LLM SDK streaming, formatting, and tool-call parsing to OpenJarvis. Allows Lumina to easily support local execution (Ollama, MLX).
- **Complexity**: Medium. Requires strict boundary definition to avoid leaking OpenJarvis objects into Lumina's core state.

### 3. Study and Rebuild Orchestration & Executor Patterns (Priority: Medium)
- **Action**: Do not import `Agent`, `Executor`, or `Manager`. Instead, study `src/openjarvis/agents/orchestrator.py` and `executor.py` to inform Lumina's native routing, RAG context injection, and state continuity mechanisms.
- **Value**: OpenJarvis's SQLite-based state is too opinionated and risks conflicting with Lumina's continuity layer. Rebuilding ensures Lumina retains total control over memory.
- **Complexity**: High (Rebuilding), but ensures long-term architectural stability.

### 4. Delegate Tool Implementations (Priority: Low)
- **Action**: Selectively import or adapter-wrap complex tools from `src/openjarvis/tools/` (e.g., Browser automation via Playwright, Shell execution).
- **Value**: Saves immense time on boilerplate implementations for multi-modal and environment-interaction tools.
- **Complexity**: Low to Medium, depending on tool dependencies.

## Summary
"Delegate execution abstraction and tool execution; Own orchestration, memory, and continuity."
