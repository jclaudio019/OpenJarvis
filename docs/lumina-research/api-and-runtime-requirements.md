# API and Runtime Requirements

## Required APIs
- Based on the `pyproject.toml` and engine implementations, the framework relies on standard REST/WebSocket connections to LLM providers if cloud usage is opted in.
- Expected to support `openai` format endpoints heavily, utilizing SDKs like `openai`, `anthropic`, and `google-genai`.

## Optional APIs
- **Tool APIs**: Various integrations depending on activated tools (e.g., Tavily, DuckDuckGo for search, GMail/Calendar via Google API).
- **Communication Channels**: Numerous optional SDKs (Telegram, Slack, Discord, Reddit, etc.) for `channel_agent.py` deployments.

## Local-only Capability Support
- The architecture is explicitly designed for "Local-first" operation.
- **Ollama**: First-class support (`engine/ollama.py`) for local model execution without cloud dependencies.
- **Hardware Acceleration**: Explicit dependencies for GPU metrics (`pynvml`, `amdsmi`) and Apple Silicon (`zeus-ml[apple]`, `mlx-lm`).

## Docker/Runtime Requirements
- For isolated sandbox execution (e.g., Code Interpreter tools), a working Docker daemon is required (`docker>=7.0`).
- WebAssembly execution (`wasmtime>=25`) is an alternative sandbox.
- Rust extensions are mentioned in the README for deep local functionalities (requires compilation/downloads).

## GPU/Model Requirements
- Local operation implies specific VRAM availability depending on the selected model size. The `model_catalog.py` lists parameters such as `min_vram_gb`.
- Local execution frameworks include VLLM, Ollama, and MLX depending on the platform (macOS/Linux).

## Cloud Dependencies
- Fully optional. If using cloud models, API keys for OpenAI, Anthropic, Google, MiniMax, or OpenRouter must be available in the environment.

## Authentication Requirements
- Cloud API keys (`OPENAI_API_KEY`, etc.).
- OAuth for Google integrations (Drive/Gmail/Calendar).
- Standard token/auth flows for communication channels (Slack tokens, Discord bot tokens).
