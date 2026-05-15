# Maintainability and Risk Analysis

## Architectural Complexity
- **High Modularity**: The use of `RegistryBase` across the system implies excellent decoupling. Adding new providers, tools, or memory backends does not require modifying core logic.
- **Complex Executor State**: The `Executor` and `Manager` modules hold significant complex logic around SQLite persistence, retries, and context compression (`LoopGuard`). Modifying or upgrading these will be high-risk for consumers depending on precise agent behavioral traces.

## Dependency Risk
- **Broad Tooling Footprint**: The `pyproject.toml` defines a vast number of optional dependencies (e.g., `playwright`, `discord.py`, `pynvml`, `pdfplumber`). This means integrating OpenJarvis wholesale carries immense dependency weight and potential vulnerability surface.
- **Vendor SDKs**: Heavy reliance on moving targets like `openai>=1.30` and `anthropic>=0.30`. Upgrades to these SDKs upstream may break OpenJarvis integrations if not carefully tracked.

## Repo Activity and Upgrade Difficulty
- As a project emerging from Stanford research initiatives (Intelligence Per Watt), the repo may prioritize experimental feature velocity (e.g., local model profiling) over strict backward compatibility in its early Alpha stages (`Development Status :: 3 - Alpha`).
- Upgrade difficulty is **Moderate to High** if Lumina deeply hooks into internal state types. If restricted via adapters to the `InferenceEngine` interface, upgrade difficulty is **Low**.

## Long-term Viability for Lumina
- **As a Core Dependency**: Using OpenJarvis as a core foundational block for Lumina is **risky**. The project's goals (Personal AI with heavy local state via SQLite) fundamentally differ from Lumina's goals as an orchestration and routing layer.
- **As an Extracted Resource**: **Highly viable**. Portions like the `model_catalog`, specific `tools/` implementations, and telemetry formats represent thousands of hours of optimization that Lumina can lift-and-shift.
