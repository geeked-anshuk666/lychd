# LychD Reference Manual & Comprehensive Architecture Guide

> **Location:** `d:/computer_science/Snayu_AI/my-reference/`  
> **Target Project:** [LychD (The Hexanomicon)](https://github.com/hexanomicon/lychd)  
> **Status:** Easy-to-understand, modular, end-to-end breakdown of the entire system.

---

## 📚 Table of Contents

1. [01 - System Overview & What is LychD](01_SYSTEM_OVERVIEW.md)
   - Plain English definition of LychD.
   - The core philosophy: Local sovereignty vs. Cloud feudalism.
   - High-level architecture in one view.
   - What is possible with LychD (practical capabilities, agent swarms, workflows).

2. [02 - The Lore vs Technical Reality (Rosetta Stone)](02_LORE_AND_TERMINOLOGY.md)
   - Decoding the mystical occult names into engineering components.
   - The Lich, Vessel, Phylactery, Animators, Ghouls, Altar, Sigil, Runes, Scrolls, etc.
   - Inner Tongue vs. Iron Tongue table.

3. [03 - Codebase Directory Breakdown](03_CODEBASE_STRUCTURE.md)
   - Complete dissection of `src/lychd/`, `clients/`, `docs/`, `tests/`, and build scripts.
   - What each folder and critical file does.
   - How the pieces interact at runtime.

4. [04 - Persistence & Memory (The Phylactery)](04_PERSISTENCE_AND_MEMORY.md)
   - PostgreSQL + `pgvector` database schema and migrations (`0001`–`0023`).
   - Advanced Alchemy, SQLAlchemy models, and session management.
   - Semantic memory (Karma), episodic runs, artifacts, and long-term storage.

5. [05 - Agent Execution & Graph Orchestration](05_AGENTS_AND_GRAPH.md)
   - Pydantic AI integration and typed agent definitions.
   - State graph execution, human-in-the-loop (HITL) consent gates, and dispatchers.
   - Asynchronous worker queues (SAQ Ghouls) and distributed jobs.

6. [06 - Inference Engines & Model Serving (Animators)](06_ANIMATORS_AND_INFERENCE.md)
   - Local engines: llama.cpp, vLLM, SGLang, ExLlamaV3/TabbyAPI.
   - Remote Portals vs. Local Soulstones.
   - Model switching, GPU allocation, PCIe P2P, and memory constraints.

7. [07 - Frontend & UI (The Altar & Clients)](07_ALTAR_AND_CLIENTS.md)
   - Svelte/SvelteKit Altar, Litestar ASGI serving, and Server-Sent Events (SSE).
   - Canvas, Loom, Nexus, Orb interfaces.
   - Multi-client ecosystem: Web, Android, Discord, Minecraft, Research bots.

8. [08 - Compositions & Capabilities (What Can You Build?)](08_COMPOSITIONS_AND_USE_CASES.md)
   - Real-world application templates: Foundry, Atelier, Keeper, Scavenger, Homestead, etc.
   - Step-by-step workflow examples.

9. [09 - Comprehensive Tech Stack & Architectural Justifications](09_TECH_STACK_AND_JUSTIFICATIONS.md)
   - Exhaustive deep-dive into every single package, runtime, framework, and database used in LychD.
   - What each tech does, why it was chosen over its alternatives (e.g. Litestar vs FastAPI, Granian vs Uvicorn, Svelte 5 vs React, SAQ vs Celery, basedpyright vs mypy, uv vs poetry).
   - Architectural trade-offs and design decisions.

10. [10 - 100 Practical, Mind-Blowing & Revenue-Generating Applications](10_100_USE_CASES_AND_APPLICATIONS.md)
    - 100 concrete, categorized real-world systems, SaaS ideas, and sovereign agent workflows you can build on LychD.
    - Spans Developer Tooling, High-Margin B2B SaaS, Deep Intelligence, Air-Gapped Legal/Medical, Media Publishing, Red Teaming, Homelab, and Gaming.

---

## 🚀 How to Read These Guides
- **If you are completely new to LychD**: Start with [01_SYSTEM_OVERVIEW.md](01_SYSTEM_OVERVIEW.md) and [02_LORE_AND_TERMINOLOGY.md](02_LORE_AND_TERMINOLOGY.md).
- **If you want to read/modify code**: Go directly to [03_CODEBASE_STRUCTURE.md](03_CODEBASE_STRUCTURE.md) and [04_PERSISTENCE_AND_MEMORY.md](04_PERSISTENCE_AND_MEMORY.md).
- **If you want to build an agent or use-case**: Read [05_AGENTS_AND_GRAPH.md](05_AGENTS_AND_GRAPH.md) and [08_COMPOSITIONS_AND_USE_CASES.md](08_COMPOSITIONS_AND_USE_CASES.md).
