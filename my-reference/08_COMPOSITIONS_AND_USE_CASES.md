# 08 - Compositions & Practical Use-Cases: What Can You Build?

In LychD, a **Composition** is a self-contained, pre-architected application suite that bundles together specific agents, tools, state graphs, database schemas, and UI surfaces for a defined problem domain.

This document details the primary built-in compositions and demonstrates the types of applications and autonomous pipelines you can build with LychD.

---

## 1. Primary Built-In Compositions

### 1. **Foundry & Workshop** (Autonomous Software Engineering)
- **Goal**: Full-lifecycle code generation, refactoring, and test automation.
- **How It Works**:
  - Ingests repository files and architectural rules (`AGENTS.md`, ADRs).
  - Decomposes tasks into sub-tasks (e.g. `test-red`, `implement`, `refactor`).
  - Executes unit tests in an isolated sub-process or container.
  - When tests fail, it parses traceback errors and automatically patches code until tests turn green.
  - Generates git commits and structured PR descriptions.

### 2. **Scavenger & Reach** (Sovereign Web Intelligence & Research)
- **Goal**: Deep, automated web research without relying on commercial search APIs.
- **How It Works**:
  - Uses autonomous browser automation or direct HTTP crawlers.
  - Scrapes technical documentation, arXiv papers, and web blogs.
  - Cleans HTML, extracts core text, and summarizes findings.
  - Embeds insights into `Karma` (`pgvector`) for permanent recall across future sessions.

### 3. **Atelier & Creation** (Creative Media & Asset Production)
- **Goal**: Multimodal asset generation and management.
- **How It Works**:
  - Coordinates ComfyUI, Stable Diffusion, or local vision pipelines.
  - Manages image prompt variations, upscaling passes, and style consistency.
  - Retains all output files in content-addressed storage with cryptographic hashes (`ArtifactRef`).

### 4. **Blockworld** (Embodied AI in Virtual Worlds)
- **Goal**: Autonomous agent embodiment in 3D Minecraft worlds.
- **How It Works**:
  - Connects to Minecraft server instances via custom bot protocols.
  - Implements spatial memory, inventory management, building planning, and cooperative play.
  - Listens to player voice in-game and responds with speech synthesized locally in real time.

---

## 2. Walkthrough: Constructing a Custom Agent Workflow

Here is a conceptual example of how you build a custom Composition in LychD using its native tools and Pydantic AI graph engine:

```python
from pydantic import BaseModel
from pydantic_ai import Agent
from lychd.domain.orchestration.schema import WorkflowGraph

# 1. Define Typed Inputs and Outputs
class ResearchInput(BaseModel):
    topic: str
    max_sources: int = 5

class ResearchSummary(BaseModel):
    title: str
    key_findings: list[str]
    confidence_score: float

# 2. Define the Agent Node with Capability Boundaries
researcher_agent = Agent(
    model="soulstone:fast_reasoning",  # Routes to local vLLM / llama.cpp
    result_type=ResearchSummary,
    system_prompt="You are an autonomous research intelligence node..."
)

# 3. Attach Sovereign Tools (Scavenger)
@researcher_agent.tool
async def fetch_web_page(url: str) -> str:
    # Uses LychD's sovereign web crawler
    from lychd.domain.web.crawler import fetch_clean_markdown
    return await fetch_clean_markdown(url)

# 4. Bind into the Phylactery for Long-Term Storage
async def persist_research(session_id, summary: ResearchSummary):
    from lychd.db.models.karma import Karma
    from lychd.domain.animation.embeddings import compute_embedding
    
    vector = await compute_embedding(summary.title + " " + " ".join(summary.key_findings))
    # Stored in PostgreSQL with pgvector
    karma_record = Karma(
        kind="fact",
        content=summary.model_dump_json(),
        embedding=vector,
        session_id=session_id
    )
    # Saved and available forever
    # Saved into the Phylactery for long-term recall
    await session.commit()
```

*(For an exhaustive breakdown of 100 practical, revenue-generating, and creative systems you can build with LychD, see [10_100_USE_CASES_AND_APPLICATIONS.md](10_100_USE_CASES_AND_APPLICATIONS.md).)*

