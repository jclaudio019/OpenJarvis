# Adapter Recommendation

## Proposed Integration Approach

Given that Lumina is acting as a cognitive orchestrator, the most effective integration approach is **not** to run OpenJarvis as a black-box daemon, but rather to use it as a **Library / API Wrapper**.

Alternatively, if strict sandboxing is required, an **MCP (Model Context Protocol) Integration** could be built over OpenJarvis's tool suite.

### Primary Recommendation: API Wrapper for Engines and Tools

Lumina should consume OpenJarvis primarily for its `InferenceEngine` abstractions and its `tools` module.

#### Recommended Interface Shape

```python
# lumina/adapters/openjarvis_engine.py

from openjarvis.engine import get_engine
from lumina.core.llm import LLMAdapter, LuminaMessage, StreamChunk

class OpenJarvisEngineAdapter(LLMAdapter):
    def __init__(self, provider_id: str):
        # Maps "openai", "ollama", "anthropic" to OpenJarvis internal engines
        self.engine = get_engine(provider_id)

    async def stream_generate(
        self,
        messages: list[LuminaMessage],
        model: str,
        tools: list[dict] = None
    ):
        # Convert Lumina formats to OpenJarvis core types
        oj_messages = self._convert_messages(messages)

        async for chunk in self.engine.stream_full(
            oj_messages,
            model=model,
            tools=tools
        ):
            yield self._convert_chunk(chunk)
```

### Minimal Integration Surface

To minimize risk and coupling, Lumina should strictly interact with:
1. `src/openjarvis/engine/`: Exclusively for sending prompt states and receiving streams/tool calls.
2. `src/openjarvis/tools/`: As a registry of callable Python functions that Lumina's own executor invokes.
3. `src/openjarvis/core/types.py`: Only for serialization/deserialization boundaries.

**Do not integrate with:**
- `src/openjarvis/agents/manager.py` (State persistence)
- `src/openjarvis/core/events.py` (Internal event bus)
- The OpenJarvis CLI.

### Maintenance Concerns
- **Dependency Drift**: OpenJarvis updates to underlying vendor SDKs (e.g., `openai`, `anthropic`) could force Lumina to resolve version conflicts.
- **Data Shape Changes**: If OpenJarvis modifies `core.types`, the Lumina adapter will break. The adapter layer must include strict validation and tests.

### Risks
- **Tight Coupling**: Relying too heavily on `InferenceEngine` might make Lumina dependent on OpenJarvis's pace of implementing new provider features (e.g., specific vision or audio modalities).
- **SQLite Locking**: If Lumina accidentally invokes parts of OpenJarvis that interact with its default SQLite DB, concurrent access issues may arise.
