# Part 3: What Currently Exists vs. What is Work-in-Progress

LychD is in an active **Pre-Alpha** development state. Understanding what is real, battle-tested code versus what is still aspirational or partially implemented saves hours of debugging time.

---

## 1. What is Fully Built, Verified & Solid in the Codebase

These modules have complete implementations, passing test suites, and database schemas:

### A. The Database & Persistence Layer (The Phylactery)
- **Status**: Production-grade and verified.
- **Details**:
  - Full PostgreSQL schema with 23 Alembic migrations (`versions/0001` through `versions/0023`).
  - Strict database-enforced state machine constraints (e.g., a `Run` marked `awaiting_consent` must link to a valid `consent_id`).
  - Advanced Alchemy / SQLAlchemy 2.0 async engine with connection pooling.
  - `pgvector` schema for semantic `Karma` records.

### B. The ASGI Web Server & Security (The Vessel)
- **Status**: Complete and functional.
- **Details**:
  - Litestar + Granian (Rust Tokio engine) configured in `src/lychd/app.py`.
  - Loopback-only security policy (`server_policy.py`) protecting against DNS rebinding, CSRF, and non-local origins.
  - Full OpenAPI 3.1 specification generation (`openapi.json`).

### C. Typed Agent Contracts & Pydantic AI Integration
- **Status**: Complete and integrated.
- **Details**:
  - Models and tool parameters are strictly typed using Pydantic v2 schemas.
  - Structured output parsing, prompt template generation, and runtime schema enforcement.

### D. Human-in-the-Loop (HITL) Consent Engine
- **Status**: Complete.
- **Details**:
  - State machine that physically pauses execution when high-risk tools are triggered.
  - Dedicated API endpoints (`/api/consents/{id}/grant`) and CLI commands to approve or deny pending actions.

### E. Developer Toolchain & Testing Infrastructure (The Forge)
- **Status**: Mature and strictly enforced.
- **Details**:
  - 6,000+ tests spanning unit, integration, and OCI container fixtures.
  - Instant formatting and linting via `ruff`, strict static typing via `basedpyright`, and dependency locking via `uv`.

---

## 2. What is Partial, Experimental, or In-Progress

These are areas where foundational code exists, but full automated operation is still being refined:

### A. Dynamic Model VRAM Lifecycle Management
- **Current State**: LychD can connect to already-running vLLM or llama.cpp servers over HTTP/OpenAI-compatible protocols.
- **In-Progress**: Dynamic on-the-fly unloading of one 70B model to load another 32B model directly from the daemon without manual operator commands.

### B. The SvelteKit Altar Web Interface
- **Current State**: Front-end skeleton with full TypeScript types generated from the backend OpenAPI schema, `@xyflow/svelte` for node graph rendering, and CSS styling.
- **In-Progress**: Full end-to-end polish of all interactive panels (Canvas, Loom, and Orb diagnostics).

### C. Multi-Node Peer Federation (Intercom / Mesh)
- **Current State**: Database models (`0023_peer_graph_custody`) and RFC 9421 cryptographic HTTP message signatures are implemented.
- **In-Progress**: Full multi-machine agent peer discovery and distributed execution across local home networks.

### D. Packaged Compositions
- **Current State**: Schemas, interfaces, and reference logic exist under `src/lychd/compositions/` (Foundry, Scavenger, Atelier, Blockworld).
- **In-Progress**: End-to-end turnkey CLI commands for deploying each composition out-of-the-box.

