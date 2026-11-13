# Litestar-Fullstack Parallels & Architectural Inheritance

> **Target Directory:** `my-reference/parallels-from-litestar-fullstack/`  
> **Source Comparison:** [Litestar Fullstack](https://github.com/litestar-org/litestar-fullstack) vs [LychD](https://github.com/hexanomicon/lychd)

---

## 🧭 Overview

LychD's Python structure was not invented from scratch; **its foundational lineage directly descends from [Litestar Fullstack](https://github.com/litestar-org/litestar-fullstack)**. In fact, LychD's own `README.md` explicitly credits Litestar Fullstack:
> *"Litestar and Granian raise and serve the Vessel; its lineage descends from Litestar Fullstack."*

Understanding `litestar-fullstack` is the ultimate Rosetta Stone for understanding LychD: LychD adopted the enterprise production architecture of `litestar-fullstack` and transformed its domain from **Multi-Tenant SaaS (Users, Teams, Billing)** into an **Autonomous AI Agent Daemon (Inference, Graphs, Vector Memory)**.

---

## 📚 Documentation Index

1. [01 - Architectural Lineage & Overview](01_ARCHITECTURAL_LINEAGE.md)
   - The origin story: How Litestar Fullstack provided the structural skeleton for LychD.
   - Core philosophy comparison: Production Web SaaS vs. Sovereign AI Agent Daemon.
   - High-level directory tree mapping.

2. [02 - Side-by-Side Component Parallels](02_SIDE_BY_SIDE_PARALLELS.md)
   - Detailed mapping table comparing every module, file, and layer.
   - Server lifespans, plugin architecture, database repositories, background worker queues, and CLI.

3. [03 - What LychD Kept vs. What It Replaced](03_WHAT_WAS_KEPT_VS_REPLACED.md)
   - What remained identical (Advanced Alchemy, Alembic, Granian, SAQ, Structlog, SvelteKit).
   - What was removed (User registration, multi-tenancy, Stripe billing, JWT cookies).
   - What was introduced (Pydantic AI, pgvector Karma, HITL Consent, Animators, Wards).

4. [04 - How to Use Litestar-Fullstack to Master & Extend LychD](04_HOW_TO_EXTEND_AND_REVERSE_ENGINEER.md)
   - Reading clean, non-obfuscated patterns in `litestar-fullstack` to debug LychD.
   - How to re-introduce multi-user accounts and teams into LychD using Litestar Fullstack code.
   - Normalizing LychD code back to standard software patterns.

