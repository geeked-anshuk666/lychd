# Issue #131 — Core backend: prove one native Run and define the Nexus contract

- **Issue:** https://github.com/hexanomicon/lychd/issues/131
- **State:** open (snapshot 2026-11-16) · **Labels:** `area:runtime`, `tracking`, `area:network`
- **Assignees:** `geeked-anshuk666` (you), `NavSrijan`
- **Linked:** #132 (Altar Core, Run design), #133 (Altar Core Bridge, Debayan), #153 (Nav's backend execution), #150 (open PR, Nav's Reach model binding), #152 (your PostgreSQL trace), #18, #8, #20, #19, #29

---

## 1. What the issue asks for (plain English)

Pick **one** Core use case (the issue suggests the Discord bot), and prove that it reaches the **native Graph** path, not just a demo fixture. Then write down the request/result contract that the Nexus UI and the Altar will rely on.

Your part (from the issue body):

1. **Restart test.** After a supported application restart against the *same PostgreSQL data*, read back the exact result of the Run.
2. **Feedback.** If the workflow supports linked human feedback, check it. If not, use a separate example and name the gap.
3. **Nexus.** With Nav, record what Nexus can still show after a restart, and what becomes *unknown*. Hand Debayan enough steps to repeat the Core check in #133.

Nav does the request-to-result trace (admission → capability grant → result → cleanup). You own the durability half of that story.

**Acceptance is not "it works on my laptop".** The issue wants each claim tagged as: proven, fixture, dated observation, or missing.

---

## 2. Prerequisites — what to know first

### Tech stack you need
| Area | What to know | Why it matters here |
|---|---|---|
| Python 3.12-ish + `uv` | `uv sync`, `uv run`, `pyproject.toml`, lockfile discipline | Setup and every test command go through `uv`/`make` |
| Litestar | Routes, dependency injection, lifespan events | The "application restart" is a Litestar lifespan boot |
| SQLAlchemy 2.0 (async) + Advanced Alchemy | Async sessions, transactions, `UUIDAuditBase` models | Runs, steps, consent, checkpoints are rows |
| Alembic | Migration files under `src/lychd/db/migrations/versions/` | Restart tests depend on schema state |
| PostgreSQL | Running a DB, roles, `psql` basics | The restart evidence is PostgreSQL-backed |
| SAQ (Simple Async Queue) | What a "Ghoul" job is, enqueue vs. run | Run execution is queued out of process |
| Pydantic AI + Pydantic Graph | Typed agents, graph nodes, "awaiting consent" state | This is what "native Graph" means |
| Podman or Docker | Rootless containers | `make test` runs inside an OCI image; there is **no host fallback** |
| Discord bot basics (optional for step 1) | Bot tokens, slash commands | The suggested use case; you can substitute a synthetic request |

### Architecture you need to understand
Read these in order (repo files; my-reference notes in brackets):

1. [CONTRIBUTING.md — Find the part you want to change](../../CONTRIBUTING.md) — "Run admission, execution, or recovery" → ADR 14 (Workers) and ADR 24 (Graph).
2. [docs/state-of-the-work.md — Persistence, execution, and consent](../../docs/state-of-the-work.md#persistence-execution-and-consent) — **the most important page for this issue.** It says what is *Partial* and what is *Not yet*. Key lines:
   - Run, step, session, consent, checkpoint shapes exist with sequence-fenced claim and settlement.
   - A **historical** two-boot Bridge Run recovery receipt exists; it is *frozen* and does not qualify current OCI tests.
   - **Not yet:** PostgreSQL and SAQ are *not* one transaction; Step events have no outbox; checkpoint-plus-consent recovery and process-crash recovery are unqualified.
3. [docs/state-of-the-work.md — Outcome matrix](../../docs/state-of-the-work.md#outcome-matrix) — one-table summary of what is Partial vs. solid.
4. [docs/adr/06-persistence.md](../../docs/adr/06-persistence.md) — persistence law.
5. [docs/adr/24-graph.md](../../docs/adr/24-graph.md) and [docs/adr/25-hitl.md](../../docs/adr/25-hitl.md) — graph and consent.
6. Your reference notes: `my-reference/04_PERSISTENCE_AND_MEMORY.md` (tables: `run`, `step`, `session`, `consent`) and `my-reference/05_AGENTS_AND_GRAPH.md` (Run lifecycle diagram).

**Caution:** `my-reference/04_…` says migrations run `0001`–`0023`. The repo now has more (up to `0027` plus dated migrations). Trust the directory listing, not that doc.

### Concepts (short glossary)
- **Run** — one workflow execution; statuses include `queued`, `running`, `awaiting_consent`, `awaiting_peer`, `done`, `failed`, `cancelled`.
- **Step** — one atomic action inside a Run.
- **Consent** — a human approval gate; a Run can pause on it and must resume after restart.
- **Checkpoint** — the saved graph position a Run resumes from.
- **Nexus** — the Altar surface that shows Run/transition state. Its contract is *what this issue must define*.
- **Core** — the minimal end-to-end path the team agreed on. See #132 and #133.

---

## 3. Do you need to summon a Lich for this issue?

**Short answer: use the test-level restart path first. Add a partial summon on your openSUSE VPS for the restart readback only.**

The VPS (3 vCPU, 6 GB RAM, no GPU) can run the application without a model. Per [the Tumbleweed guide](../../docs/deployments/tumbleweed/index.md), that tests startup and authenticated PostgreSQL/queue readback, which is the restart evidence this issue needs. It cannot answer chat, and the guide says container lifecycle and remote model inference are still blocked. Chat or model-dependent Run steps need a fixture or a clearly labelled stand-in, and must not be reported as answer quality.

Reasons (from the repo, not guesses):

- [docs/summoning.md](../../docs/summoning.md) says, in its own warning, that **summoning does not work end to end yet**. It is a pre-alpha host acceptance procedure, and it requires:
  - Linux with cgroup v2, rootless Podman **≥ 5.4**, Quadlet, systemd user manager with `loginctl enable-linger`;
  - an NVIDIA GPU with CDI;
  - a tool-capable GGUF model placed at `~/models/first-model.gguf`;
  - `localhost/lychd:dev` built from the same checkout.
- Windows is an **unsupported** hosting platform for the deployment path (ADR 08). You are on Windows 11, so a full summon is out of reach here.
- The issue's acceptance is about **persistence and restart**, which the repo already exercises with focused tests. Summoning is the *later* "host acceptance" step.

**Recommended path:**

| Stage | What to do | When |
|---|---|---|
| A | Run the focused PostgreSQL/restart tests in the OCI test image (below) | Now |
| B | Reproduce one Run through the native Graph path with a fixture model, then restart | Now / with Nav |
| C | Only after #131 names a supported profile: a host-level summon on a Linux box with NVIDIA hardware | Later; coordinate with the founder |

If you do want to try a summon later, follow `docs/summoning.md` gates in order and treat each gate as a separate observation. Do not claim a gate you did not record.

---

## 4. How to proceed — step by step

### Step 0 — Set up (once)
```bash
cd D:\computer_science\Snayu_AI\lychd      # your checkout
make install                                 # host CLI tooling in .venv (uv.lock)
make help                                    # list the targets
```
Make sure Podman (rootless) or Docker is installed and running. `make test` will refuse to fall back to the host.

### Step 1 — Pin the revision and profile
- Record `git rev-parse HEAD`, your Python/uv versions, and the Postgres image/version.
- Note the profile you use (for the issue's "supported application profile" requirement).
- Write this in a trial record **outside** the checkout (the repo says private host material stays outside).

### Step 2 — Find the existing restart evidence (read, don't write yet)
Open these and read the test names and what they assert:
- `tests/integration/test_db_run_ledger_pg.py` — Run ledger on PostgreSQL.
- `tests/integration/test_database_authority_pg.py` — roles and schema authority.
- `tests/integration/test_production_wiring.py::test_production_wiring_real_factory_over_postgres_survives_second_boot` — **the closest existing "survives a restart" test.** This is the one to learn from.
- `tests/integration/test_local_bridge_process_restart.py` — independent-process restart.
- `tests/unit/domain/cortex/test_ledger.py` — ledger unit tests.

### Step 3 — Run the focused checks
```bash
# Backend lane from CONTRIBUTING (runtime + orchestration):
make test PYTEST_TARGETS="tests/unit/system/services/test_runtime.py tests/unit/domain/orchestration"

# Agentic lane (Bridge workflow, delegation):
make test PYTEST_TARGETS="tests/agents tests/unit/domain/delegation tests/unit/domain/cortex/test_delegate_resume.py"

# PostgreSQL receipts (needs a disposable PostgreSQL / daemon socket):
make test-containers
```
Record pass/fail/blocked for each. If a command can't run on your machine, say so — don't mark it passed.

### Step 4 — Reproduce one Run with a restart
1. Choose **one** public or synthetic request (no private data).
2. Admit it through the native path (the Bridge workflow or the one Nav picked).
3. Record Run ID, workflow revision, and the exact result.
4. Stop the application cleanly, start it again on the **same** PostgreSQL data.
5. Read back the Run and its result by ID. Record whether it matches exactly.
6. Also try:
   - a **repeated** identical request (expected: idempotent, not a duplicate effect);
   - one **refusal** or **unknown** case (expected: explicit refusal or unknown state, never silent success).
7. Record what is *unknown* after a crash (what the repo says is not yet qualified).

### Step 5 — Define the Nexus contract with Nav (#132)
Agree the request/result envelope and write it down:
- caller identity;
- operation and inputs;
- task and revision identity;
- execution destination (local / peer);
- audience of the result;
- outcomes: `success`, `refused`, `unavailable`, `unknown`.

Keep it as a short markdown table in the issue or a PR. Do not invent a new schema; propose changes for founder review.

### Step 6 — Hand off
- Give **Debayan** (#133) the exact steps to repeat Step 4.
- Give **Chirayu/Kiran** (#132) the states the UI can show (running, awaiting consent, done, failed, unknown).
- Prepare a single evidence packet: demo/trace, changed contracts, failure observations, checks run, next decision.

### Step 7 — Respect the review rules
- Keep **at most one** review-ready implementation slice awaiting the founder.
- Shared API types, checkpoint identity, and effect settlement need review from the adjacent lane (CONTRIBUTING, "Claim one contract in an issue before changing it").
- Follow the branch and PR rules in CONTRIBUTING ("Branches and everyday collaboration") — the current phase is team integration through `dev`.

---

## 5. Things that could go wrong (watch for these)

- **Outbox gap:** Step events have no outbox, so a crash between a DB write and a queue event can lose or duplicate work. Say this out loud; it is listed as "Not yet" in State of Work.
- **Consent resume:** a Run stuck in `awaiting_consent` must resume after restart. If it doesn't, that is a finding, not a flake.
- **PR #150 overlap:** Nav's open PR #150 changes Reach model binding and Portal dispatch opt-in. Don't assume the Reach path is stable while it's open.
- **Historical receipts are frozen:** they prove what *once* happened on a dated profile; they do not prove the current build.

---

## 6. Checklist (copy into the issue when working)

- [ ] Revision, profile, and tool versions recorded
- [ ] Existing restart tests read and understood
- [ ] Focused `make test` / `make test-containers` results recorded (pass/fail/blocked)
- [ ] One Run admitted, restarted, and read back exactly
- [ ] Repeat request is idempotent
- [ ] One refusal or unknown case observed
- [ ] Nexus states after restart recorded (including "unknown")
- [ ] Nexus contract agreed with Nav and #132
- [ ] Steps handed to Debayan (#133)
- [ ] Evidence packet posted for founder review
