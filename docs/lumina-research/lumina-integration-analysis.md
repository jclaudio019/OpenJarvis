# Lumina Integration Analysis

## Capabilities Lumina Should Use Directly
- **Engine Abstractions**: OpenJarvis has a highly developed, robust abstraction for LLM inference engines (`InferenceEngine`) supporting streaming, tool calling, and multi-provider compatibility (OpenAI, Anthropic, Google, MiniMax, Ollama, MLX). Lumina can leverage this via adapter to avoid rewriting provider integrations.
- **Tools Library**: The extensive `src/openjarvis/tools` directory (e.g., browser automation, shell execution, DB querying) contains production-ready implementations. Wrapping these as discrete capabilities would rapidly accelerate Lumina.

## Capabilities That Should Remain External
- **Agent Loops/Executor Patterns**: The specific implementation of `OrchestratorAgent` and the `Executor` tick model are tightly coupled to the OpenJarvis SQLite-based manager state. Lumina, as a cognitive orchestrator, should implement its own execution and continuity models.
- **Local State/Manager**: `Manager` (`manager.py`) assumes direct control over local SQLite databases for tracking memory, learning logs, and message queues. Lumina should manage its own memory model and state persistence.

## Capabilities That Should Eventually Be Extracted
- **Model Catalog (`model_catalog.py`)**: The comprehensive specs (cost, context length, active parameters) represent valuable, hard-coded domain knowledge that could be extracted into a standalone configuration repository or module.
- **Telemetry and Tracing Types**: The types defined in `core/types.py` (e.g., `TelemetryRecord`, `TraceStep`) are excellent designs for evaluating and tracking agent performance.

## Tightly Coupled and Risky
- **Agent Manager/Database Layer**: Direct use of the OpenJarvis state system risks architectural conflicts with Lumina's continuity layer.
- **LoopGuards and Heuristics**: The `LoopGuard` is coupled to the structure of OpenJarvis's `Message` queue and string-based context manipulation, which could clash with Lumina's abstract context management.

## Stable and Reusable
- **Registry Pattern**: The generic `RegistryBase` is a simple, highly reusable dependency injection/discovery pattern.
- **Engine Discovery**: Dynamic model and engine discovery paths (`_discovery.py`) appear stable and useful for detecting local hardware capabilities.
