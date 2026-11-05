# Part 1: The Rosetta Stone of Lore Jargons

LychD intentionally wraps its architecture in an occult dark fantasy metaphor (*"The Dark Arts of LLMs"*). The overarching theme is **necromancy**: an immortal AI daemon living on your hardware that cannot be permanently destroyed because its memory and state are anchored to a database.

This document serves as the direct dictionary translating every mythic and occult term into its real-world software engineering equivalent.

---

## Complete Translation Dictionary

| Lore Jargon (Inner Tongue) | Plain English Translation | Real Software Equivalent | What it actually does in the code |
|---|---|---|---|
| **The Lich** | The Autonomous Daemon / Agent Identity | Background System Process | The top-level Python application managing memory, agency, and orchestration (`src/lychd/domain/`). |
| **The Phylactery** | The Permanent Database | PostgreSQL 16+ with `pgvector` | The durable storage. If your computer crashes, reboots, or the process dies, the "soul" (state, conversation, memory) is preserved here. |
| **The Vessel** | The Web Server / Runtime Container | Litestar + Granian (ASGI) in Podman | The runtime execution environment hosting the HTTP API, SSE token streams, and background tasks. |
| **Runes** | Configuration Declarations / Rules | Pydantic Schemas & JSON / TOML configs | Declarative rules defining agent goals, allowable tools, token budgets, and safety boundaries. |
| **Scrolls** | Workflow Pipelines | State Machine / Directed Acyclic Graph (DAG) | A predetermined multi-step workflow script (e.g. Step 1: Research $\rightarrow$ Step 2: Code $\rightarrow$ Step 3: Test). |
| **Weaver / Spellweaver** | Workflow Orchestrator | Graph Engine / DAG Scheduler | The orchestrator (`src/lychd/domain/orchestration/`) that connects agents, tools, and steps to execute a Scroll. |
| **Animators** | Inference Providers / Model Backends | vLLM, llama.cpp, SGLang, ExLlamaV3 | The model engines that load weights and generate text/code tokens. |
| **Soulstone** | Local Inference Runtime | Local GPU/CPU Engine | A model running directly on your own hardware (zero data egress, private, zero API fees). |
| **Portal** | Remote Inference API | OpenAI / Anthropic / Groq endpoint | An external API gateway used when local VRAM is unavailable. |
| **Animus** | Active Model Weights | Loaded LLM in VRAM | The neural network currently residing in memory producing tokens. |
| **Ghouls** | Background Task Workers | SAQ (Simple Async Queue) + Redis | Asynchronous background workers that execute heavy jobs (scraping, indexing, bulk refactoring) without freezing the web server. |
| **Altar** | Frontend Web Dashboard | SvelteKit + Vite Web App | The web browser UI where you chat with agents, inspect graph execution trees, and approve actions. |
| **Sigil** | API Key / Bearer Token | Cryptographic Auth Token | The authorization credential that grants access to the HTTP API and CLI. |
| **Karma** | Semantic / Vector Memory | `pgvector` Embeddings | Long-term associative memory. Chunks of past lessons, facts, and mistakes embedded into vector space for similarity search. |
| **Sediment** | Execution Audit Trail | Relational Event Log (`step` & `run` tables) | An immutable history of every turn, tool call, prompt, and output ever made. |
| **Ward** | Data Privacy & Redaction Firewall | Data Loss Prevention (DLP) Filter | Middleware that strips API keys, passwords, and PII before text is sent to an external model. |
| **Covenants** | Architectural Rules & Decisions | Architecture Decision Records (ADRs) | The 43 foundational design specifications in `docs/adr/`. |
| **Compositions** | Packaged Multi-Agent Apps | Pre-built Application Templates | Bundles of agents + tools + DB schemas tailored for specific tasks (e.g., Foundry = coding, Scavenger = web scraping). |
| **Forge** | Developer Tooling & Build System | `uv`, `Ruff`, `basedpyright`, `Pytest` | The development tools used to maintain, lint, test, and package LychD. |
| **Iron** | Underlying Hardware & OS | Bare metal / Linux OS | The physical GPUs, RAM, Linux kernel, systemd, and Btrfs filesystems. |

