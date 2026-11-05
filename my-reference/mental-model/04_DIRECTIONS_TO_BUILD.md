# Part 4: Directions in Which You Can Build and Extend LychD

Because LychD provides an uncommonly strong foundation—**$0 token costs on local open weights**, **permanent relational and vector memory (`pgvector`)**, **asynchronous task queues (SAQ Ghouls)**, and **tamper-proof audit trails**—you can take this repository in several high-impact directions.

---

## Direction 1: The Autonomous Homelab & System Administration Sentry
**Concept**: Turn LychD into a private, sovereign system administrator that runs 24/7 on your local workstation or server.

### What to Build:
- Connect the **Ghouls** (SAQ workers) to system metrics (GPU VRAM, disk health, CPU temperatures).
- Implement automated Btrfs snapshot management: create instantaneous snapshots before running system updates or large builds, and roll back automatically if health tests fail.
- Build a smart storage sentry that cleans docker caches, identifies orphan files, and organizes downloads.

### Why LychD is Best for This:
- Completely private: no system diagnostics, hostnames, or file names are ever transmitted to a cloud vendor.
- `systemd` and Podman Quadlet native: runs cleanly as a background Linux service.

---

## Direction 2: Sovereign Autonomous Coding Appliance (The Foundry)
**Concept**: An agent that receives a bug report or feature request, clones the target repo into an isolated container, and iteratively writes code until tests pass.

### What to Build:
- Flesh out `src/lychd/compositions/workshop/` and `foundry/`.
- Create a Pydantic AI state graph that:
  1. Inspects failing test tracebacks.
  2. Identifies relevant source files using semantic search across the codebase.
  3. Applies candidate edits.
  4. Runs pytest in a temporary container.
  5. If tests fail, feeds the error back to the model and repeats up to $N$ iterations.
  6. Submits a clean git branch and commit once all tests turn green.

### Why LychD is Best for This:
- **Zero Token Fees**: Iterating 20 times on a complex bug with a 70B model costs nothing locally. On Claude or OpenAI APIs, that single debugging loop could cost several dollars.
- **HITL Consent**: The agent cannot commit to `main` or push to remote remotes without you clicking "Grant" on the Altar web UI.

---

## Direction 3: Deep Sovereign Web Research & Knowledge Archive (The Scavenger)
**Concept**: A self-hosted competitor to NotebookLM or Perplexity that lives permanently on your PC.

### What to Build:
- Extend `src/lychd/domain/web/` and `src/lychd/domain/archive/`.
- Provide a CLI or web input where you paste 10 URLs, a topic keyword, or a directory of 50 research PDFs.
- Ghouls download, parse, and clean the content in the background, chunk the text, compute vector embeddings, and save them into the `karma` table.
- Build a synthesis endpoint that queries the vector database and generates a structured research dossier with exact source citations.

### Why LychD is Best for This:
- The `pgvector` schema (`src/lychd/db/models/karma.py`) is already built and indexed.
- Research memory never expires; your local agent retains knowledge across months and years of sessions.

---

## Direction 4: Persistent Embodied Agents & Gaming NPCs (Blockworld)
**Concept**: Autonomous entities that inhabit virtual worlds (such as Minecraft or Discord communities) with persistent memory, spatial awareness, and real-time voice.

### What to Build:
- Hook up the `clients/minecraft/` client to an active Minecraft server.
- Bind the bot's perception (inventory, surrounding block voxel data, player chat) into Pydantic AI tools.
- Connect local text-to-speech (e.g. Kokoro or Piper) to stream audio when players approach the bot.

### Why LychD is Best for This:
- Standard LLM game bots forget players after 10 minutes when context windows wipe. LychD's `Karma` table allows the bot to remember whether a player helped or attacked them days ago.

