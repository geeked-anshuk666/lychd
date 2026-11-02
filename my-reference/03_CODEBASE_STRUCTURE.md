# 03 - Codebase Directory Breakdown & File Guide

This document breaks down the repository tree of `lychd`, explaining what each folder does, its primary responsibilities, and key files you need to know.

---

## 1. Top-Level Repository Structure

```
Snayu_AI/lychd/
├── .agents/              # Agent rules, workflows, and task scopes (xDDD methodology)
├── clients/              # Frontends & integration clients (Web SvelteKit, Discord, Android, Minecraft)
├── docs/                 # The Hexanomicon documentation site (Zensical / Markdown)
├── examples/             # Concrete example configurations and usage scripts
├── scripts/              # Helper maintenance scripts (migrations, DB init, dev tools)
├── src/                  # Main Python application package (src/lychd/)
├── tests/                # Test suite (unit, integration, contract, runtime fixtures)
├── Containerfile         # Production container definition (rootless Podman)
├── Containerfile.tests   # OCI test runner container definition
├── Makefile              # Master automation entrypoint (lint, test, build, format)
├── pyproject.toml        # uv / Python dependency definitions & tool configuration
├── ROADMAP.md            # The 5-Book founding trajectory
└── zensical.toml         # Documentation site builder configuration
```

---

## 2. Core Python Package (`src/lychd/`)

The heart of the system lives in `src/lychd/`. It is structured into clearly separated architectural layers:

### A. Root Modules
- `app.py`: The ASGI application factory. Configures Litestar, sets up dependency injection, mounts HTTP routers, OpenAPI plugins, and registers lifecycle hooks (startup / shutdown).
- `__main__.py`: CLI entrypoint. Invoked when running `python -m lychd` or the `lychd` binary.
- `developer.py`: Development mode helpers and hot-reload harnesses.

---

### B. Core Architectural Subpackages

#### 1. `domain/` (Business Logic & Cognitive Capabilities)
The pure business logic and agent definitions, completely decoupled from HTTP frameworks.
- `animation/`: Manages inference runtimes. Houses providers for **vLLM**, **llama.cpp**, **SGLang**, and **TabbyAPI/ExLlama**, handling model loading, prompt dispatch, token streaming, and capability checks.
- `archive/`: Memory archiving logic, retention policies, and semantic compaction.
- `delegation/`: Manages agent-to-agent delegation, sub-agent invocation, and task assignment.
- `orchestration/`: Multi-agent orchestration, state graphs, and pipeline scheduling.
- `privacy/`: Privacy boundaries, token masking, and confidential data redaction before sending tokens to models or external portals.
- `intercom/`: Inter-agent communication protocols and message schemas.
- `artifact_custody.py`: Content-addressed storage and custody tracking for generated artifacts (images, code, documents) via SHA-256 digests.
- `web/`: Sovereign web interaction (scraping, browser control, DOM parsing).

#### 2. `db/` (The Phylactery: PostgreSQL & Advanced Alchemy)
Everything related to relational data and vector storage.
- `engine.py` & `factory.py`: SQLAlchemy async engine and connection pool creation.
- `sessions.py`: Async session scopes and transaction managers.
- `models/`: SQLAlchemy 2.0 ORM models mapping runs, turns, agent states, artifacts, memories, and users.
- `migrations/`: Alembic database migration scripts (`versions/0001_...` through `versions/0023_...`).
- `archive_semantic.py`: Vector embeddings search (`pgvector`) implementation for semantic memory (Karma).
- Specific repositories: `intercom.py`, `creation.py`, `scout.py`, `scavenger.py`, `ward.py`, `blockworld.py`.

#### 3. `interface/` (API & Web Transport)
Connects the domain logic to external callers.
- `server.py`: Granian / Litestar server configuration and socket binding.
- `server_policy.py`: Origin, CORS, CSRF, loopback safety, and host validation guards.
- `api/`: REST and Server-Sent Events (SSE) route handlers.
  - Endpoints for sending messages, triggering workflows, polling task runs, and retrieving artifacts.
- `discord/`: Native Discord bot interface for interacting with the Lich via chat commands.
- `web/`: Serves static assets and SvelteKit frontend artifacts.

#### 4. `ghouls/` (Background Worker Infrastructure)
Asynchronous task execution using **SAQ** (Simple Async Queue).
- Handles long-running jobs that should not block the web server (e.g. vector embedding computation, web crawling, code refactoring passes).
- Worker process configuration, queue registration, retry backoffs, and job cancellation.

#### 5. `compositions/` (Packaged Application Templates)
Ready-to-use compositions of agents, workflows, and tools tailored to specific domains:
- `foundry/`: Software development agent (scaffolding, testing, code editing).
- `atelier/`: Visual and creative asset generation.
- `scavenger/`: Web scraping, document ingestion, and synthesis.
- `blockworld/`: Minecraft bot integration with game-world spatial cognition.

#### 6. `cli/` (Command Line Interface)
Subcommands accessible through the terminal:
- `commands.py`: Main CLI parser.
- `binding.py`: Configuring host addresses, ports, and loopback rules.
- `lifecycle_view.py`: Daemon lifecycle control (start, stop, status).
- `companion_view.py`: Inspecting agent states and memories.
- `deletion.py`: Safe data pruning and retention enforcement.

#### 7. `system/` & `config/`
- `system/constants.py`: System-wide constants, version strings, and default timeouts.
- `config/`: Pydantic-based configuration settings loaded from environment variables and `.env` files.

---

## 3. Clients Subdirectory (`clients/`)

LychD provides multiple consumer interfaces in addition to the CLI:
- `clients/web/`: The primary Svelte / SvelteKit UI (**The Altar**). Features modern chat, graph visualization, and artifact viewers.
- `clients/discord/`: Discord bot client allowing group interaction with agents.
- `clients/android/`: Mobile client package for monitoring and controlling the daemon.
- `clients/minecraft/` & `clients/minecraft-voice/`: Minecraft mod and bot integrations allowing the Lich to exist as an NPC in virtual worlds.
- `clients/research/`: Specialized testing scripts and evaluation harnesses.
