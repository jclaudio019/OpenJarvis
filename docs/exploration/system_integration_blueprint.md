# System Integration Blueprint: Advanced Autonomous Capabilities

## 1. Current Architecture Trace

### 1.1 Entry Points and Interfaces
OpenJarvis currently intercepts user inputs via channels (e.g., Discord, Slack, CLI) through a unified messaging system. The primary interface for orchestrating this logic is the `JarvisSystem` defined in `src/openjarvis/system/core.py`.

Channels are integrated via `wire_channel`, which hooks into incoming messages. A user query flows through `JarvisSystem.ask()`, delegating the core logic to a lazy-loaded `QueryOrchestrator`.

### 1.2 Orchestration and Intent Detection
The `QueryOrchestrator` (`src/openjarvis/system/orchestrator.py`) handles standard query formulation, tool aggregation, and agent dispatch.
- **Intent Detection:** Uses naive regex matching (e.g., matching "good morning" to the `morning_digest` agent). This acts as a rigid, hard-coded intent gateway.
- **Agent Dispatching:** If an agent is targeted, `_run_agent` delegates to specific subclasses of `BaseAgent` registered in `AgentRegistry`. If no agent is detected, execution falls back directly to the Inference Engine (the language model).

### 1.3 State and Modularity
- **Rigid Components:** The conversational intent routing relies on hard-coded heuristics. The context collection and injection (`inject_context`) is a single-pass process before standard LLM inference. Execution loops (like tools or multi-hop agents) run synchronously within individual agent subclasses (like `morning_digest`) without a global mechanism to pause, reflect, or request human signatures.
- **Modular Components:** The backbone is inherently decoupled via the `RegistryBase` pattern (`src/openjarvis/core/registry.py`). There are independent registries for Models, Engines, Memories, Agents, Tools, Skills, etc. This design acts as an ideal foundation for injecting new skills and agent executors. Tools are injected as lists of class instances into agent scopes contextually.

---

## 2. Modular Integration Design

To implement reasoning-first multi-agent orchestration, we propose the following decoupled directory layout. These modules seamlessly interoperate with the existing `RegistryBase`.

```text
src/openjarvis/
├── planning/                   # Upfront Context-Gathering & Human-on-the-Loop
│   ├── __init__.py
│   ├── prober.py               # Generates clarifying questions (What, Why, How)
│   ├── plan_generator.py       # Outputs structural macro-plans
│   └── approval_gate.py        # Halts execution pending human signatures
├── execution/                  # Hybrid Task Execution
│   ├── __init__.py
│   ├── engine.py               # The Step-by-Step State Validator
│   └── validation/             # Localized validators against expected schemas
│       ├── __init__.py
│       └── data_schema.py
├── memory/
│   ├── dialectic/              # Post-turn dialectic reflection & insights
│   │   ├── __init__.py
│   │   ├── reflection.py       # Background error & log analyzer
│   │   └── memory_commit.py    # Merges structural insights to long-term DB
│   └── vector_store/           # (Existing semantic search logic)
└── skills/
    ├── interfaces/             # Abstract Wrappers for specialized external interfaces
    │   ├── base_autonomous.py
    │   └── environment.py
    └── dynamic_loader.py       # Autonomously registers new tools without core edits
```

### 2.1 Dynamic Skills Interface
We introduce an isolated `BaseAutonomousSkill` class in `skills/interfaces/` that extends the generic `BaseTool`. External plugins (like shell access or deep data research) can inherit this and be mapped into the `SkillRegistry`. This maintains separation of concerns, keeping specialized code away from `QueryOrchestrator`.

---

## 3. The Core Loop Specification

We transition the current synchronous architecture into a multi-phase, state-aware flow.

### Flow Diagram:

1. **User Command Received:** An instruction is intercepted by the Channel Bridge.
2. **Clarification Loop (Context Gathering):**
   - The *Planning Prober* evaluates the ambiguity of the command.
   - *If broad:* Yields execution, asks the user for What/Why/How boundaries. (Repeat until satisfied).
3. **Plan Generation & Signature:**
   - The *Plan Generator* outputs a structured markdown or JSON Step-by-Step execution macro-plan.
   - The *Approval Gate* pauses the system. It will not proceed without a cryptographic or explicit conversational approval signature from the user.
4. **Hybrid Executor (Step Execution & Self-Validation):**
   - The executor handles Step $N$.
   - **Step Execution:** Performs data fetching, tool use, or computation.
   - **Self-Validation:** Cross-checks outputs against expected schema constraints natively before moving on.
   - *If Validation Fails:* Re-evaluates context natively.
5. **Failure Reflection (Dialectic Insights):**
   - If a step completely halts or yields an unrecoverable logic error, the *Dialectic Reflection Engine* processes the tool traces and logs in the background. It surfaces underlying misconceptions in system understanding.
6. **Post-Turn Memory Commits:**
   - Insights, user preferences deduced during clarification, and successful step templates are committed as robust graph or associative links to the Long-Term Memory state, improving context for the next turn.
