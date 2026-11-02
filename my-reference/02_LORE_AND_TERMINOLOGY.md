# 02 - Lore and Terminology: The Rosetta Stone

LychD uses an extensive mythical and occult vocabulary inspired by necromancy and dark fantasy (*"The Dark Arts of LLMs"*). While atmospheric, this terminology can be disorienting when reading the codebase.

This document serves as the **Rosetta Stone**, mapping every mythic concept ("Inner Tongue") directly to its real-world software engineering counterpart ("Iron Tongue").

---

## 1. Core Mythological Metaphor

> In fantasy lore, a **Lich** is an immortal sorcerer who binds their soul into an indestructible vessel called a **Phylactery**. Even if the Lich's physical body is destroyed, it reconstitutes itself from its Phylactery and cannot truly die.

In LychD:
- **The Lich** = The unified AI entity and its long-running agentic identity.
- **The Phylactery** = The PostgreSQL + `pgvector` database where state, episodic memories, and embeddings are preserved permanently.
- **The Vessel** = The runtime application process (`systemd` container / ASGI server) hosting the Lich. If the container or server reboots, the Lich resurrects by reloading its state from the Phylactery.

---

## 2. Complete Glossary & Technical Mapping

| Occult / Lore Term (Inner Tongue) | Technical Term (Iron Tongue) | Codebase Location | Explanation |
|---|---|---|---|
| **The Lich** | Autonomous Agent System / Daemon Identity | `src/lychd/domain/lich/` | The top-level software identity and cognitive coordination system. |
| **Vessel** | ASGI Web Application & Process Host | `src/lychd/app.py`, `system/` | The runtime environment; runs Litestar + Granian inside a Podman container. |
| **Phylactery** | Database & Persistence Layer | `src/lychd/db/`, `domain/persistence/` | PostgreSQL with `pgvector` storing runs, events, artifacts, and memories. |
| **Animator** | LLM Inference Engine / Provider | `src/lychd/domain/animation/` | An endpoint that serves model inference (llama.cpp, vLLM, SGLang, ExLlama). |
| **Soulstone** | Local Inference Engine | `src/lychd/domain/animation/` | A local model process running directly on the host GPUs/CPUs. |
| **Portal** | Remote API Provider | `src/lychd/domain/animation/` | An external endpoint (e.g. OpenAI API, external vLLM server). |
| **Animus** | Loaded Model Weights | N/A (Concept) | The active neural network currently loaded into memory producing tokens. |
| **Ghouls** | Background Worker Queue | `src/lychd/ghouls/` | [SAQ](https://github.com/tobymao/saq) asynchronous workers executing queued background tasks. |
| **Altar** | Frontend Web Application | `clients/web/` | Svelte / SvelteKit UI for interacting with agents, inspecting graphs, and chatting. |
| **Hexanomicon** | Project Documentation Site | `docs/` | The documentation site generated via Zensical / MkDocs. |
| **Sigil** | Authentication Token / Bearer Key | `src/lychd/interface/http/` | The cryptographic authorization credential for API requests. |
| **Rune** | Intent / Configuration Specification | `src/lychd/domain/` | A declarative configuration defining agent goals, tools, or limits. |
| **Scroll** | Workflow / Graph Pipeline Definition | `src/lychd/domain/graph/` | A defined execution workflow, state graph, or agent DAG. |
| **Karma** | Semantic / Associative Vector Memory | `src/lychd/domain/memory/` | Chunks and embeddings stored in `pgvector` for semantic similarity retrieval. |
| **Sediment** | Historical Storage / Event Ledger | `src/lychd/db/models/` | Immutable append-only audit trail of turns, tool calls, and state transitions. |
| **Sepulcher** | The Core Architecture Core | `docs/sepulcher/` | Architectural grouping containing Vessel, Phylactery, Lich, and Animators. |
| **Divination** | User Interaction & Perception | `docs/divination/` | Modules relating to user interfaces, web observation, and external world sensing. |
| **Compositions** | Packaged Multi-Agent Applications | `src/lychd/compositions/` | Pre-built agent application templates (e.g., Foundry, Atelier, Scavenger). |
| **Forge** | Developer Tooling & Build System | `Makefile`, `pyproject.toml` | The linters, type-checkers (Ruff, basedpyright), testing tools, and compilers. |
| **Iron** | Underlying Hardware & OS | Bare metal / Linux OS | The physical GPUs, RAM, Linux kernel, systemd, and Btrfs filesystems. |

---

## 3. The Lifecycle of an Action: In Lore vs. In Code

To understand how a user request flows through these occult abstractions:

```
[User at the Altar]
         │ (Submits request via Svelte Web UI)
         ▼
[Through the Sigil]
         │ (HTTP Bearer token validated by Litestar ASGI middleware)
         ▼
[Received by the Vessel]
         │ (Dispatched to the Lich cognitive engine)
         ▼
[Consults the Phylactery]
         │ (Loads historical state + Karma vector search from PostgreSQL)
         ▼
[Spins a Scroll via Agent Graph]
         │ (Executes Pydantic AI state graph nodes)
         ▼
[Summons an Animator]
         │ (Sends prompts/tokens to vLLM or llama.cpp for inference)
         ▼
[Enlists Ghouls for Labor]
         │ (Heavy long-running jobs pushed to SAQ async background workers)
         ▼
[Records the Sediment]
         │ (Commits final response, execution trace, and artifacts to DB)
         ▼
[Reflected on the Altar]
           (Streams token output & state graph events via SSE back to browser)
```
