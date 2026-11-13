# 03 - What LychD Kept vs. What It Replaced

Understanding the deliberate architectural trade-offs made when LychD branched off from Litestar Fullstack allows you to see the true purpose of every module.

---

## 1. What LychD Kept Intact (The Shared Kernel)

LychD recognized that Litestar Fullstack had solved all the hard "plumbing" problems of modern async Python development. It kept:

1. **The Modern Web Kernel**:
   - `Litestar 2.x` with automatic OpenAPI 3.1 generation.
   - `Granian` ASGI server (Rust-based HTTP/1.1 and HTTP/2 handling via Tokio).
2. **The Relational Persistence Stack**:
   - Async `SQLAlchemy 2.0` engine.
   - `Advanced Alchemy` for clean generic repositories, eliminating boilerplate CRUD queries.
   - `Alembic` for database schema migrations.
3. **The Task Queue**:
   - `litestar-saq` for async background tasks (avoids the heavy footprint of Celery).
4. **Structured Logging**:
   - `structlog` for contextual, machine-readable JSON logging.
5. **The Frontend Stack**:
   - SvelteKit + Vite + TypeScript.
   - `openapi-typescript` compiling `openapi.json` into type-safe client calls (`openapi-fetch`).
6. **The Developer Tooling (The Forge)**:
   - `uv` for ultra-fast dependency management and lockfiles.
   - `ruff` for linting and formatting.
   - `basedpyright` for strict static type checking.

---

## 2. What LychD Stripped Out (Removed SaaS Complexity)

`litestar-fullstack` was designed to sell subscriptions to multiple business teams. LychD is designed for **local sovereignty** (one operator controlling private AI agents). Consequently, LychD removed:

1. **Multi-Tenancy & Teams**:
   - Removed `app/domain/teams/` (Organizations, invitations, member roles).
2. **User Registration & Passwords**:
   - Removed user sign-up flows, password hashing with Argon2, email verification tokens, and session cookies.
   - Replaced by a single local cryptographic key (**The Sigil**) and loopback-only binding (`127.0.0.1`).
3. **Billing & Payments**:
   - Removed Stripe webhooks, subscription tiers, and payment customer tracking.
4. **Email Dispatching**:
   - Removed SMTP / transactional email templates (Jinja2 emails).

---

## 3. What LychD Added (The Sovereign AI Layer)

To transform the stripped SaaS skeleton into an autonomous agent platform, LychD introduced:

1. **Vector Memory & Embeddings (`pgvector`)**:
   - Added the `pgvector` PostgreSQL extension to enable cosine similarity search.
   - Created the `karma` table for long-term associative memory (*"Sediment, Not Dump"*).
2. **Pydantic AI Integration**:
   - Embedded `pydantic-ai-slim` to handle typed LLM calling, schema parsing, and prompt templates.
3. **Human-in-the-Loop (HITL) State Machine (ADR 25)**:
   - Built the `Run` $\rightarrow$ `Step` $\rightarrow$ `Consent` relational state engine. If an agent calls a dangerous tool, the database freezes the run at `awaiting_consent` until a human clicks approve.
4. **Model Inference Adapters (Animators)**:
   - Added adapters for local GPU runtimes (**vLLM**, **llama.cpp**, **SGLang**, **ExLlamaV3/TabbyAPI**) and remote OpenAI-compatible endpoints.
5. **Data Loss Prevention Firewall (The Ward)**:
   - Middleware that intercepts outbound prompts, redacting API keys, passwords, and private data before text leaves the local machine.
6. **Sovereign Web Crawler (Scavenger)**:
   - Embedded headless browser and HTTP scraping tools to acquire web documents without paid search APIs.
7. **Graph Visualization UI (Loom & Canvas)**:
   - Added `@xyflow/svelte` to the SvelteKit frontend to visually inspect agent state graphs and node transitions in real time.

