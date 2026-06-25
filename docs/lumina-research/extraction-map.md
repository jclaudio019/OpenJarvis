# Extraction Map

## Exact Files/Modules Most Useful for Lumina

1. **`src/openjarvis/core/registry.py`**
   - **Value**: Clean, decoupled dependency injection framework using generic typing. Useful for building Lumina's plugin architecture.
2. **`src/openjarvis/engine/cloud.py` & `src/openjarvis/engine/ollama.py`**
   - **Value**: High-quality, tested wrappers over complex vendor APIs handling edge cases for streaming, tool calls, and model formatting.
3. **`src/openjarvis/core/types.py`**
   - **Value**: Defines telemetry, tracing, and structured message formats. Highly informative for designing Lumina's cognitive data schemas.
4. **`src/openjarvis/intelligence/model_catalog.py`**
   - **Value**: Hardcoded knowledge base of LLMs, contexts, and capabilities. Excellent to port directly.

## Low-Dependency Components
- **`src/openjarvis/core/events.py`**: A simple event bus implementation.
- **`src/openjarvis/core/types.py`**: Pure data classes without complex external logic.

## Reusable Abstractions
- **`InferenceEngine` base class (`src/openjarvis/engine/_base.py`)**: A strong interface definition for LLM interaction that normalizes outputs across varied backends.
- **Tool Definitions (`src/openjarvis/tools/_stubs.py`)**: Schema standard for defining inputs, descriptions, and execution context for capabilities.

## Orchestration Logic Worth Studying
- **`src/openjarvis/agents/orchestrator.py`**: Explains how to parse non-native tool calls (using regex for `THOUGHT/TOOL/INPUT`), how to run tools in parallel via `ThreadPoolExecutor`, and manage turn limits.
- **`src/openjarvis/agents/executor.py`**: The `Agent Tick` loop provides excellent insights into error recovery (e.g., handling empty model outputs), state finalization, and RAG context injection prior to generation.

## Memory Structures Worth Studying
- **`src/openjarvis/memory/` (Implied via RAG in executor)**: How contexts are scored and dynamically prepended to the user input space when a loop begins.

## Planner/Executor Boundaries
- OpenJarvis sharply separates *decision making* (`Agent.run`) from *execution mapping* (`Executor`). The agent yields structured intent, and the executor handles the physical implementation and state updating. This boundary is critical for a cognitive orchestrator like Lumina to study.
