# 04 - How to Use Litestar-Fullstack to Reverse Engineer & Extend LychD

Having a clean copy of `litestar-fullstack` next to `lychd` inside `Snayu_AI/` gives you an immense superpower.

Whenever LychD's occult metaphors or dense abstractions feel overwhelming, you can use `litestar-fullstack` as a clear, plain-English reference manual.

Here is how you can practically use this relationship to debug, simplify, or extend LychD.

---

## 1. Reverse-Engineering Confusing LychD Code

### Example: Understanding LychD's Database Repositories
- If you look at `lychd/src/lychd/db/` and get confused by how sessions and transactions are handled:
- Open `litestar-fullstack/src/py/app/domain/accounts/services.py` and `db/engine.py`.
- You will see the standard, textbook implementation of an `SQLAlchemyAsyncRepository`:
  ```python
  class UserService(SQLAlchemyAsyncRepository[User]):
      model_type = User
  ```
- You now immediately understand that `KarmaRepository` or `RunRepository` in LychD is doing the exact same thing—just storing agent runs instead of user accounts!

---

## 2. Re-Introducing Multi-User & Teams to LychD

Currently, LychD is strictly single-user (local operator only). But what if you want to deploy LychD on a remote VPS and let multiple team members log in and share agents?

Because LychD inherits Litestar Fullstack's database and session design, you can easily **port multi-tenancy back in**:

1. Copy `src/py/app/domain/accounts/` from `litestar-fullstack` into `lychd/src/lychd/domain/accounts/`.
2. Copy the `User` and `Team` models into LychD's database models.
3. Run `alembic revision --autogenerate -m "add users and teams"`.
4. Now your LychD agents can have owners, team workspaces, and role-based permissions!

---

## 3. Normalizing LychD Back to Plain English

If you plan to build a commercial product or enterprise system on top of your LychD fork, you may not want your team or clients to see necromancy terms like *The Lich* or *Phylactery*.

You can easily rename the layers using standard Litestar Fullstack terminology:

| LychD Lore Term | Normalized Software Term (from Litestar Fullstack) |
|---|---|
| `The Lich` | `AgentDaemon` / `AgentOrchestrator` |
| `Phylactery` | `AgentDatabase` / `PersistenceService` |
| `Vessel` | `Server` / `AppServer` |
| `Animators` | `ModelProviders` / `InferenceEngines` |
| `Ghouls` | `TaskWorkers` / `BackgroundQueue` |
| `Altar` | `AgentDashboard` / `WebUI` |
| `Sigil` | `ApiKey` / `BearerToken` |
| `Karma` | `VectorMemory` / `SemanticStore` |
| `Sediment` | `EventLog` / `ExecutionHistory` |
| `Ward` | `PrivacyFirewall` / `DataFilter` |

---

## 4. Comparing Best Practices Across Both Projects

Whenever you are unsure where to place a new feature in LychD:
1. **Does it touch the database?** Look at `litestar-fullstack/src/py/app/db/models/`.
2. **Does it add a new API route?** Look at `litestar-fullstack/src/py/app/domain/<feature>/controllers.py`.
3. **Does it add a background task?** Look at `litestar-fullstack/src/py/app/lib/worker.py`.
4. **Does it add a CLI command?** Look at `litestar-fullstack/src/py/app/cli/commands.py`.

By following the clean patterns laid out in Litestar Fullstack, you can ensure any code you write for LychD remains robust, type-checked, and architecturally sound!

