# Architecture Overview

## High-Level Architecture
OpenJarvis is built around a modular and extensible architecture, heavily leaning on the decorator-based `RegistryBase` pattern to decouple core abstractions from their implementations. The key subsystems are organized into components like `core`, `agents`, `tools`, `engine`, and `memory`, connected through dynamic discovery and configuration. It acts as an opinionated framework for building personal, multi-turn AI agents prioritizing local-first execution.

### Key Subsystems
- **Core (`src/openjarvis/core`)**: Contains the foundational types (`Trace`, `Message`, `Conversation`), the event bus (`events.py`), the settings definitions (`config.py`), and the central registry pattern (`registry.py`).
- **Agents (`src/openjarvis/agents`)**: Implements distinct agent loops. Key classes like `OrchestratorAgent`, `MonitorOperative`, and `DeepResearch` implement various reasoning/execution flows (ReAct, function calling, thought-action-observation). The `Manager` oversees state, while the `Executor` manages agent ticks and context handling.
- **Engine (`src/openjarvis/engine`)**: Provides an abstraction layer (`InferenceEngine`) over underlying LLM runtimes (Ollama, LiteLLM, OpenAI Compat, Google, Anthropic).
- **Intelligence (`src/openjarvis/intelligence`)**: Handles model specifications, catalogs (`model_catalog.py`), routing policies, and configuration of model endpoints.
- **Tools (`src/openjarvis/tools`)**: A broad repository of tools (browser automation, code execution, storage management, DB querying) exposed to agents through standardized interfaces.

## Execution Model
The execution model is centered around an "Agent Tick". The `Executor` class wraps individual model invocations, handling:
1. Context compression (via `LoopGuard` or other constraints).
2. Memory retrieval (prepending retrieved knowledge to the context).
3. The generation request (`agent.run()`).
4. Result finalization: parsing the response, updating agent state, writing responses to persistent storage, and recording telemetry/budget usage.

Agent outputs are strictly typed to support tool execution (`ToolCall` -> `ToolResult`). Agents can be invoked in continuous monitoring loops or on-demand chat modes.

## Memory Model
Memory in OpenJarvis is tiered:
1. **Short-Term/Working Memory**: Kept in the `Conversation` queue, often limited by a sliding window.
2. **Retrieval-Augmented Generation (RAG)**: Document chunking, indexing, and semantic search are integrated directly into the executor pipeline.
3. **Summary Memory/Digest Storage**: Persistent storage of past facts and structured observations to provide continuity across sessions.

## Planning & Orchestration Structure
Planning is handled by specific agent types (e.g., `OrchestratorAgent`). The orchestrator can parallelize tool calls (if supported) or run them sequentially.
- Responses from the model are typically structured (either native function calling or text-based like `THOUGHT:` / `TOOL:` / `INPUT:` / `FINAL_ANSWER:`).
- `LoopGuard` mechanisms prevent cyclic or repetitive failures.
- State persistence and tracking are deeply integrated, allowing long-horizon autonomous behavior.
