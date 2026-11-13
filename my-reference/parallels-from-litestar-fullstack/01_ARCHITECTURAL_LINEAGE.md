# 01 - Architectural Lineage: From SaaS Boilerplate to AI Agent Daemon

## 1. The Heritage of LychD

When developers first look at LychD, its dense necromantic vocabulary (*The Lich, Phylactery, Vessel, Animators, Ghouls, Grimoire*) makes it look like an alien, bizarre architecture.

However, beneath the fantasy lore lies a **classic, battle-tested Python production pattern**: the architecture of **[Litestar Fullstack](https://github.com/litestar-org/litestar-fullstack)**.

`litestar-fullstack` is the official reference boilerplate developed by the Litestar organization to showcase how to build enterprise-scale web applications in Python. The creator of LychD took this entire production framework and re-engineered its domain logic to serve local AI agents.

---

## 2. High-Level Comparison: The Core Transition

```
┌────────────────────────────────────────┐
│     Litestar Fullstack Boilerplate     │
├────────────────────────────────────────┤
│ • Litestar + Granian (Rust Tokio ASGI) │
│ • Advanced Alchemy + PostgreSQL        │
│ • SAQ Background Job Workers           │
│ • SvelteKit + Vite Web UI              │
│ • Domain: Accounts, Teams, Billing     │
└──────────────────┬─────────────────────┘
                   │
                   │ (LychD retains the entire infrastructure,
                   │  replaces SaaS domain with AI Cognition)
                   ▼
┌────────────────────────────────────────┐
│             LychD Daemon               │
├────────────────────────────────────────┤
│ • Litestar + Granian (The Vessel)      │
│ • Advanced Alchemy + pgvector (Phylac.)│
│ • SAQ Background Workers (The Ghouls)  │
│ • SvelteKit + Vite Web UI (The Altar)  │
│ • Domain: Pydantic AI, Graphs, Models  │
└────────────────────────────────────────┘
```

---

## 3. Top-Level Directory Tree Comparison

Notice how both repositories organize their code into identical layered boundaries:

### Litestar Fullstack (`litestar-fullstack/src/py/app/`)
```
app/
├── server/               # ASGI server setup, Granian plugins, core configuration
├── lib/                  # Shared services (settings, logging, crypto, workers)
├── db/                   # Database engine, models, and Alembic migrations
├── domain/               # Business logic modules:
│   ├── accounts/         # Users, passwords, authentication
│   ├── teams/            # Multi-tenant teams, invitations
│   └── system/           # Health probes, OpenAPI checks
├── cli/                  # Click / Rich command line management
└── __main__.py           # CLI and server runner entrypoint
```

### LychD (`lychd/src/lychd/`)
```
lychd/
├── app.py / server/      # ASGI server setup, Granian plugins, loopback policies
├── lib/ / config/        # Shared services (settings, logging, constants)
├── db/                   # Database engine, models, and Alembic migrations
├── domain/               # Cognitive logic modules:
│   ├── animation/        # LLM inference backends (vLLM, llama.cpp)
│   ├── orchestration/    # Pydantic AI state graph execution
│   └── cortex/           # State transitions, ledger, and memory
├── ghouls/               # SAQ background task runner (runs.py)
├── cli/                  # Click / Rich command line management
└── __main__.py           # CLI and server runner entrypoint
```

---

## 4. Key Takeaways

1. **Zero Reinvention of the Wheel**: LychD did not invent its own web server, database layer, or background worker queue. It inherited Litestar Fullstack's proven stack.
2. **Architectural Maturity**: Because it inherits Litestar Fullstack, LychD automatically benefits from strict type safety (`basedpyright`), high-throughput async processing (`Granian`), and structured database transactions (`Advanced Alchemy`).
3. **The Simplification Secret**: If any file in LychD's server, database, or worker code feels confusing, you can open the matching file in `litestar-fullstack` to see the original, un-mystified, standard Python version!

