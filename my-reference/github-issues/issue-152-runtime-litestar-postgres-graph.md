# Issue #152 — Runtime foundation: trace one Litestar/PostgreSQL task into Graph

- **Issue:** https://github.com/hexanomicon/lychd/issues/152
- **State:** open (snapshot 2026-11-16) · **Labels:** `area:runtime`, `area:host`
- **Assignee:** `geeked-anshuk666` (you) — leads the PostgreSQL work
- **Partner:** `NavSrijan` — traces the execution/failure/cleanup path
- **Linked:** #131 (same Core request), #132 (Run contract), #133, #153 (Nav), #20 (VPN/Tether side quest), #151 (merged Tether PR), #23 (artifacts), #26 (consent), #30 (memory), #34/#62 (training, later), #29 (Scrolls), #19 (A2A), #40 (SDLC)

---

## 1. What the issue asks for (plain English)

Pick **one supported application profile** and trace **one** task end to end:

> Litestar admission → required PostgreSQL state and transactions → native workflow/Graph → retained result.

You must:

1. **Pin** a source revision and a supported configuration/profile.
2. **Reproduce** one database-backed behavior and one failure or lifecycle case. State what persists after restart, what is unknown, and what is substituted or unavailable.
3. **Verify linked feedback** against the exact result it assesses (on the existing Creation feedback path, as the reference).
4. **Agree the request/result envelope** with Nav and #132.
5. **Record one reusable implementation pattern** and one bounded next repair — with enough steps for another developer to reproduce.

The issue is explicit: **understand the existing foundations and review shared-contract changes before multiplying implementations.** Don't rebuild a whole subsystem.

---

## 2. Prerequisites — what to know first

### Tech stack
| Area | What to know | Used for |
|---|---|---|
| Python + `uv` | Lockfile, `uv run`, editable installs | All tests and the app run through it |
| **Litestar** | ASGI app, lifespan hooks, DI, route handlers | Admission is an HTTP/Litestar call |
| **SQLAlchemy 2.0 async** | `AsyncSession`, transactions, `select`/`insert` | Every write is a transaction |
| **Advanced Alchemy** | Base models (`UUIDAuditBase`), repository helpers | Models in `src/lychd/db/models/` |
| **Alembic** | Revision files, upgrade/downgrade, autogenerate caveats | `src/lychd/db/migrations/versions/` |
| **PostgreSQL 16+** | Roles, privileges, `psql`, transactions, isolation | Real state lives here |
| **asyncpg** | Async driver, JSON/JSONB codecs | State of Work says production codecs round-trip `json` and JSONB |
| **pgvector** | Vector columns (for Karma/memory; less central here) | Schema includes it; not the focus of this issue |
| **SAQ** | Queue jobs, workers, enqueue/settle | "Ghouls" run work out of process |
| **Pydantic AI / Pydantic Graph** | Typed agents and graph nodes | The native Graph |
| Podman or Docker | Rootless containers, image builds | `make test` runs inside an OCI image |
| Linux (for real hosts) | systemd user units, cgroups | Only needed for host acceptance, not for this issue's first pass |

### Architecture you need to understand
1. [CONTRIBUTING.md — Start with three development lanes](../../CONTRIBUTING.md#start-with-three-development-lanes) — the **Backend** lane is yours: *runtime actuator: observation → effect → settlement → reopened admission or containment* (`src/lychd/system/services/runtime.py`).
2. [docs/state-of-the-work.md — Phylactery first-light persistence](../../docs/state-of-the-work.md#phylactery-first-light) — the central persistence status. Read the **Boundary — Not yet** paragraph twice:
   - PostgreSQL and SAQ are **not one transaction**.
   - Step events **lack an outbox**.
   - Memory-profile vs. PostgreSQL repository parity is **incomplete**.
   - No general retention/compaction path yet.
   - Persistent same-boot containment failure has **no durable watchdog**.
3. [docs/state-of-the-work.md — Graph stasis and consent re-admission](../../docs/state-of-the-work.md#graph-stasis-consent) and [Delegated agent execution](../../docs/state-of-the-work.md#delegated-agent-execution).
4. [docs/adr/06-persistence.md](../../docs/adr/06-persistence.md), [docs/adr/11-backend.md](../../docs/adr/11-backend.md), [docs/adr/14-workers.md](../../docs/adr/14-workers.md), [docs/adr/24-graph.md](../../docs/adr/24-graph.md).
5. Source entry points (verify they still exist before relying on them):
   - `src/lychd/db/engine.py`, `src/lychd/db/sessions.py`, `src/lychd/db/factory.py`
   - `src/lychd/db/models/run.py`, `step.py`, `consent.py`, `session.py`
   - `src/lychd/db/migrations/versions/` (first-light: `0001_phylactery_first_light.py`)
   - `src/lychd/system/services/runtime.py`
   - `src/lychd/ghouls/runs.py` (SAQ worker)
6. The issue's reference example: the Creation feedback PostgreSQL store `src/lychd/db/creation.py` (confirmed present) and the vertical test `tests/integration/test_creation_postgres_vertical.py` (confirmed present). Migrations `0016_creation_investigation.py` and `0017_creation_feedback.py` define its schema.
7. Note: a migration `0004_run_delivery_outbox.py` exists, while State of Work says Step events lack an outbox. Read both before claiming either — they may refer to different tables.

### Concepts
- **Admission** — the moment a request is accepted and its Run row is written.
- **Settlement** — the moment a Run's final state is written and its job is marked done/failed.
- **Sequence-fenced claim** — a worker only settles a Run if it still owns the expected sequence number; this stops two workers corrupting one Run.
- **Reconciliation on startup** — after a restart, leftover `running` rows are inspected and resolved.
- **Lifespan** — Litestar's startup/shutdown hooks, where migrations and queue wiring run.

### Mental model to hold
```
HTTP request ─► Litestar handler ─► [PostgreSQL txn: Run=queued] ─► SAQ enqueue
                                                                     │  (NOT the same txn)
SAQ worker ─► load Run ─► [txn: running] ─► Graph step ─► [txn: step row] ─► … ─► [txn: done]
                                                                     │
restart ─► lifespan ─► reconcile rows left in `running` / `awaiting_*`
```
The gap between "PostgreSQL txn" and "SAQ enqueue" is exactly where the issue's failure case should live.

---

## 3. Do you need to summon a Lich for this issue?

**Not for the full rite, but a partial summon on your openSUSE VPS fits the restart step.**

The VPS (3 vCPU, 6 GB RAM, no GPU) can run the application without a model. [The Tumbleweed guide](../../docs/deployments/tumbleweed/index.md) says that tests startup and authenticated database/queue readback, which is what Step 5 below needs. It cannot answer chat, and container lifecycle and remote model inference are still blocked. Use the source tests and a local PostgreSQL for the first pass; use the VPS for the restart readback only.

- The issue's acceptance is about the **runtime and database path**, which the repo already tests without a GPU.
- [docs/summoning.md](../../docs/summoning.md) requires Linux, rootless Podman ≥ 5.4, Quadlet, NVIDIA CDI, a GGUF model, and `localhost/lychd:dev`. It states that **the full sequence has no maintained real-host receipt yet**.
- Your machine is Windows 11, which is an **unsupported** hosting platform for the deployment path.
- The issue itself says "Other runtime work is tracked in #18 and #8" — full runtime acceptance is *their* job, not this issue's.

**When to revisit:** only when #152 names a supported profile that needs a real host, and the founder decides the PC topology (#131 mentions a sovereign-peer vs. same-Lich decision).

---

## 4. How to proceed — step by step

### Step 0 — Set up
```bash
cd D:\computer_science\Snayu_AI\lychd
make install
make help
```
Install Podman (rootless) or Docker. `make test` will not fall back to the host.

### Step 1 — Pin the profile
Record in a trial file **outside the checkout**:
- `git rev-parse HEAD`, `uv --version`, Python version, PostgreSQL image/version;
- which configuration and profile you're testing;
- the Run ID and workflow revision you'll use.

### Step 2 — Read the foundation (don't write code yet)
Read in this order and write a one-paragraph summary of each in your notes:
1. `src/lychd/db/factory.py` — how the database and repositories are built.
2. `src/lychd/db/models/run.py` — the Run state machine and its fields.
3. `src/lychd/db/migrations/versions/0001_phylactery_first_light.py` and `0008_wait_owner_evidence.py`.
4. `tests/integration/test_db_run_ledger_pg.py` and `tests/integration/test_production_wiring.py::test_production_wiring_real_factory_over_postgres_survives_second_boot` — the existing "survives a restart" evidence.
5. `src/lychd/system/services/runtime.py` — observation → effect → settlement.

### Step 3 — Run the backend lane check
```bash
make test PYTEST_TARGETS="tests/unit/system/services/test_runtime.py tests/unit/domain/orchestration"
```
If you change persistence, also run the PostgreSQL receipts:
```bash
make test-containers        # needs a Docker-compatible daemon; see Makefile target
```
Record every result as pass / failed / blocked / unknown.

### Step 4 — Trace one task (Nav partners on this)
1. Choose one **public or synthetic** request.
2. Trace it from Litestar admission to the Graph step to the retained result. Note each **transition owner** (who writes the row, who settles it).
3. Distinguish three things explicitly:
   - **process availability** — is the app up?
   - **application readiness** — can it accept work against the DB?
   - **authority to execute** — is the caller allowed to run this?
4. Write it as a sequence diagram (the Pattern: mermaid works in Markdown).

### Step 5 — Reproduce one database-backed behavior and one failure
Suggested pair (both must be recorded, even if one fails):
- **Behavior:** admit a Run, read back its row and result by ID after a clean restart with the **same** PostgreSQL data.
- **Failure:** interrupt the worker mid-Run (or stop the app while a Run is `running`), restart, and observe what reconciliation does.

For each, record:
- what **persists** after restart;
- what is **unknown**;
- what is **substituted** (e.g., a fixture model, a stubbed capability).

Do **not** claim "crash-safe". State of Work says process-crash and checkpoint-plus-consent recovery are **unqualified**.

### Step 6 — Verify linked feedback (if the workflow supports it)
- Find where feedback is stored and which **exact** result it refers to.
- Confirm a feedback record points to the exact candidate or revision it assesses.
- Keep feedback about *quality* separate from *authorization to act* (#26) and from *evidence that the action completed*.
- Retry the same request: it should be **idempotent**. A changed or stale reference should be **refused**. An unauthorized read should be **refused**.

If the workflow does not support feedback, say so and use the existing Creation feedback path as a *separate* example — name both.

### Step 7 — Agree the envelope and the next repair
- With Nav and #132, agree the envelope: caller, operation/inputs, task and revision identity, execution destination, audience, and outcomes (`success`/`refused`/`unavailable`/`unknown`).
- Pick **one** bounded next repair (for example: close one "Not yet" item from State of Work). Do not pick a multi-step rewrite.
- Keep at most **one** review-ready implementation slice awaiting the founder.

### Step 8 — Hand off the evidence packet
Include:
- the trace diagram;
- changed contracts (if any);
- failure observations;
- checks run and their results;
- the next decision for the founder.

Give another developer the exact commands so they can reproduce Steps 3–5.

---

## 5. Scope boundaries (don't go past these)

- **Not in scope:** the full PC/Tether topology (#20). A VPN/tunnel does not provide application admission (the issue says so explicitly).
- **Not in scope:** rebuilding PR #151 (merged Tether) or qualifying lifecycle by passing planner tests.
- **Not in scope:** Linux binding, CLI, recovery and rollback as *growth directions*, unless a real case selects them.
- **Side quest only:** Atelier (#162) and framework reuse for #155.

---

## 6. Things that could go wrong

- **Outbox gap** — a crash between the DB commit and the SAQ enqueue can lose or duplicate work. Document it; don't hide it.
- **Stale migration docs** — `my-reference/04_…` says `0001`–`0023`; the repo has more. Read the directory.
- **Not one transaction** — don't assume a rollback undoes a queued job.
- **Startup reconciliation** — a Run left in `running` after restart must be reconciled, not silently reset. If it isn't, that's the finding.

---

## 7. Checklist

- [ ] Revision, profile, and tool versions pinned (outside the repo)
- [ ] Foundation files read and summarized
- [ ] `make test` backend-lane result recorded
- [ ] `make test-containers` result recorded (or marked blocked)
- [ ] One task traced with transition owners named (with Nav)
- [ ] Process availability / readiness / authority distinguished
- [ ] One behavior and one failure reproduced; persisted / unknown / substituted recorded
- [ ] Linked feedback verified against its exact result (or gap recorded)
- [ ] Shared envelope agreed with Nav and #132
- [ ] One bounded next repair chosen; one slice awaiting review at most
- [ ] Evidence packet and reproduction steps handed off
