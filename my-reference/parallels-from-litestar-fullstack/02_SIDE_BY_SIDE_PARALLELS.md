# 02 - Side-by-Side Component Parallels

This document provides a line-by-line, file-by-file comparison of the components in **Litestar Fullstack** and their direct counterparts in **LychD**.

---

## 1. Master Parallel Comparison Table

| Architectural Layer | Litestar Fullstack (`src/py/app/`) | LychD (`src/lychd/`) | Purpose & Implementation Details |
|---|---|---|---|
| **ASGI Application Factory** | `server/core.py` (`create_app()`) | `app.py` (`create_app()`) | Instantiates the `Litestar()` object, registers OpenAPI config, CORS/CSRF middleware, plugins, and route handlers. |
| **Server Lifespan & Startup Hooks** | `server/core.py` | `interface/web/lifespan.py` | Manages startup and shutdown hooks (initializing the DB pool, establishing worker queue connections, checking hardware). |
| **Rust Tokio HTTP Runner** | `litestar-granian` plugin | `litestar-granian` plugin | Embeds Granian as the default ASGI server, providing 2x–3x faster request processing than Uvicorn. |
| **Settings & Environment** | `lib/settings.py` | `config/` (`components.py`) | Pydantic v2 `BaseSettings` parsing environment variables and `.env` files into strongly typed Python objects. |
| **Structured Logging** | `lib/log.py` | `lib/log.py` | `structlog` integration producing machine-readable JSON logs in production and pretty colored terminal output in dev. |
| **Database Connection & Engine** | `db/engine.py` & `lib/deps.py` | `db/engine.py` & `db/sessions.py` | Async SQLAlchemy 2.0 engine setup, connection pooling, and request-scoped session injection via Litestar DI. |
| **Base ORM Models** | `advanced_alchemy.base.UUIDAuditBase` | `advanced_alchemy.base.UUIDAuditBase` | Auto-generates UUID primary keys, `created_at`, and `updated_at` timestamps for every database table. |
| **Schema Migrations** | `db/migrations/` | `db/migrations/` | Alembic migration scripts evolving relational tables across versions. |
| **Asynchronous Worker Queue** | `lib/worker.py` (`litestar-saq`) | `ghouls/runs.py` (`litestar-saq`) | [SAQ](https://github.com/tobymao/saq) async queue for executing tasks out-of-process without freezing the HTTP server. |
| **Command Line Interface (CLI)** | `cli/` (`commands.py` + Click/Rich) | `cli/` (`commands.py` + Click/Rich) | Terminal management subcommands (`run`, `migrate`, `user`, etc.). |
| **Frontend Framework** | `src/js/web/` (SvelteKit + Vite) | `clients/web/` (SvelteKit + Vite) | Svelte frontend compiled into static files and served by Litestar. |
| **API Type Synchronization** | `openapi-typescript` + `openapi-fetch` | `openapi-typescript` + `openapi-fetch` | Reads Litestar's `openapi.json` and auto-generates TypeScript definitions (`openapi.d.ts`) on build. |

---

## 2. In-Depth Technical Deep Dive by Layer

### A. The ASGI Server Core
- **In Litestar Fullstack**:
  ```python
  # src/py/app/server/core.py
  def create_app() -> Litestar:
      return Litestar(
          route_handlers=domain_routes,
          plugins=[alchemy_plugin, granian_plugin, saq_plugin],
          dependencies=dependencies,
          ...
      )
  ```
- **In LychD**:
  ```python
  # src/lychd/app.py
  def create_app() -> Litestar:
      return Litestar(
          route_handlers=[api_router, web_router],
          plugins=[alchemy_plugin, granian_plugin, saq_plugin],
          middleware=[ServerPolicyMiddleware],  # Added loopback safety
          ...
      )
  ```
*Key Takeaway:* LychD's `app.py` is almost an exact replica of `core.py`, with added middleware for loopback safety and Pydantic AI agent routes.

---

### B. Database Repositories & Advanced Alchemy
Both codebases abandon traditional Django-style or raw SQLAlchemy queries in favor of the **Repository Pattern** provided by `advanced-alchemy`:
- You create an entity model inheriting from `UUIDAuditBase`.
- You create an `SQLAlchemyAsyncRepository[Model]` to handle queries.
- You inject the repository into route handlers via Litestar dependencies.

In Litestar Fullstack, this is used for:
- `UserRepository`, `TeamRepository`, `RoleRepository`
In LychD, this exact pattern is used for:
- `RunRepository`, `StepRepository`, `ConsentRepository`, `KarmaRepository`

---

### C. Background Queue (SAQ Ghouls)
Both codebases configure SAQ through `litestar-saq`:
- Background jobs are simple `async def` functions registered to a queue name.
- When an HTTP request comes in that requires heavy work, it calls `queue.enqueue("job_name", **kwargs)`.
- In Litestar Fullstack, SAQ handles sending invite emails and processing Stripe webhooks.
- In LychD, SAQ handles running Pydantic AI state graph nodes, executing web crawls, and computing vector embeddings.

