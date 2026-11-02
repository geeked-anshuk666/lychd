# 04 - Persistence & Memory: The Phylactery

In LychD, memory is not treated as an ephemeral context window dump. Following **ADR 06 (Persistence)** and **ADR 27 (Memory)**, state is managed under the doctrine of **"Sediment, Not Dump"**—an immutable, layered, audit-proof persistence engine called **The Phylactery**.

This document explains the database stack, schema design, memory hierarchy, and migrations.

---

## 1. The Persistence Technology Stack

- **RDBMS**: PostgreSQL 16+
- **Vector Extension**: `pgvector` for embedding storage and similarity indexing.
- **ORM & Toolkit**: [Advanced Alchemy](https://github.com/litestar-org/advanced-alchemy) built on top of **SQLAlchemy 2.0 (Async)**.
- **Migrations**: **Alembic** managing incremental schema changes (versions `0001` through `0023`).
- **Connection Management**: Async engine with pooling defined in `src/lychd/db/engine.py` and session lifecycles in `src/lychd/db/sessions.py`.

---

## 2. The Memory Architecture: "Sediment, Not Dump"

Instead of cramming the entire history into the LLM context window (which degrades attention and explodes token costs), LychD decomposes memory into distinct layers:

```
┌────────────────────────────────────────────────────────┐
│ 1. Working Memory (Active Context Window)             │
│    - Current turn, system prompt, tool definitions.   │
├────────────────────────────────────────────────────────┤
│ 2. Episodic Runs & Steps (The Sediment)                │
│    - Tables: `run`, `step`, `session`.                 │
│    - Step-by-step history of past agent decisions.     │
├────────────────────────────────────────────────────────┤
│ 3. Semantic Memory (Karma / pgvector)                  │
│    - Table: `karma`.                                  │
│    - Long-term associative memories, facts, mistakes.  │
├────────────────────────────────────────────────────────┤
│ 4. Cold Archive & Artifact Custody                     │
│    - Tables: `archive`, `creation`, `ward`.            │
│    - Content-addressed SHA-256 artifacts, documents.   │
└────────────────────────────────────────────────────────┘
```

---

## 3. Core Database Models (`src/lychd/db/models/`)

All models inherit from `UUIDAuditBase` (providing an auto-generated UUID primary key, `created_at`, and `updated_at` timestamps).

### A. Execution & Runs
1. **`Session` (`session.py`)**:
   - Represents a persistent conversation or user project workspace.
   - Links runs, memories, and user interactions together.
2. **`Run` (`run.py`)**:
   - Represents a single workflow or agent graph execution.
   - Key attributes:
     - `status`: `queued`, `running`, `awaiting_consent`, `awaiting_peer`, `done`, `failed`, `cancelled`.
     - `workflow_name`: The specific agent graph executed.
     - `intent`: JSONB object holding the user's initial objective.
     - `consent_id`: Foreign key to `consent` table if blocked on human approval.
3. **`Step` (`step.py`)**:
   - Atomic action within a `Run` (e.g. tool call, LLM prompt generation, result parsing).
4. **`Consent` (`consent.py`)**:
   - Human-in-the-Loop approval requests. Stores the proposed high-risk action, status (`pending`, `granted`, `rejected`), and human audit notes.

### B. Long-Term Memory & Artifacts
1. **`Karma` (`karma.py`)**:
   - Semantic long-term memory row.
   - `kind`: `note`, `mistake`, `fact`, `preference`.
   - `content`: Plain-text memory snippet.
   - `embedding`: Vector representation (managed by `pgvector`).
   - `meta`: JSONB metadata tags for filtering before vector search.
2. **`Creation` (`creation.py`) & Artifacts**:
   - Tracks generated deliverables (code files, documents, synthesized media).
   - Content-addressed: every artifact is stored and indexed by its SHA-256 digest (`ArtifactRef`).
3. **`Ward` (`ward.py`) & Privacy**:
   - Data protection policies, confidentiality tags, and egress restrictions. Ensures sensitive memories are never forwarded to unvetted external inference portals.

---

## 4. Migrations (`src/lychd/db/migrations/`)

LychD uses Alembic to maintain clean database versioning:
- Current HEAD is migration `0023` (native peer graph custody).
- Migrations span:
  - Initial tables & auth (`0001`–`0005`)
  - Run substrate, steps, and consent states (`0006`–`0010`)
  - Vector memory & Karma tables (`0011`–`0014`)
  - Intercom, companion memory, and privacy boundaries (`0015`–`0022`)
  - Distributed multi-peer graph records (`0023`).

To run or verify migrations:
```bash
# Executed via the Makefile / uv
make migrate
```
