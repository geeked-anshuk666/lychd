# 06 - Inference Engines & Model Serving: The Animators

In LychD lore, an **Animator** is an entity capable of breathing cognitive life ("Animus") into the Lich's body. In engineering terms, an Animator is an **LLM inference engine, model server, or API provider**.

This document explains how LychD connects to models, manages GPU VRAM, switches weights dynamically, and isolates inference workloads.

---

## 1. Local Soulstones vs. Remote Portals

LychD categorizes model providers into two strict classes:

| Class | Definition | Governance & Authority |
|---|---|---|
| **Soulstone (Local)** | An inference engine running directly on the local machine's GPUs/CPUs. | **Full Lifecycle Control**: LychD can start, stop, monitor VRAM, switch models, and send prompts. Safe for private and confidential data. |
| **Portal (Remote)** | An external API endpoint (e.g. cloud vLLM cluster, OpenAI-compatible endpoint). | **Inference Only**: LychD cannot start/stop the host. Requests must pass through privacy masking filters (`Ward`) to prevent accidental leaks. |

---

## 2. Supported Local Inference Engines

LychD is built to adapt to whatever local hardware you have available, natively integrating with the four leading open-source inference runtimes:

### 1. **vLLM** (The Workhorse for High-Throughput GPU Inference)
- **Best For**: Multi-user concurrency, high token throughput, batched inference, AWQ/GPTQ models.
- **Features Supported**: Continuous batching, PagedAttention, KV cache management, multi-GPU tensor parallelism.

### 2. **llama.cpp** (The Flexible GGUF Engine)
- **Best For**: Consumer PCs, mixed CPU/GPU offloading, Apple Silicon, fast model switching.
- **Features Supported**: GGUF format, partial layer offloading to VRAM, low RAM footprint, fast startup times.

### 3. **SGLang** (Radix-Powered Structured Generation)
- **Best For**: Complex agent graphs, repeated multi-turn prompts, structured JSON schema output.
- **Features Supported**: RadixAttention (tree-based KV-cache sharing across multiple branching agent paths), dramatically speeding up parallel agent evaluations.

### 4. **ExLlamaV3 / TabbyAPI** (Extreme Quantization for Consumer GPUs)
- **Best For**: Running massive models (e.g. 70B parameters) on consumer GPUs (e.g. dual RTX 3090/4090) using EXL2/EXL3 quantizations with near-zero latency.

---

## 3. The Model Factory & Capability Router (`src/lychd/domain/animation/`)

LychD abstracts model differences through a unified capability router:

- **`capabilities.py`**: Declares what each model can do:
  - `supports_tools`: Can the model parse and execute Pydantic AI function calls?
  - `supports_vision`: Can it accept image inputs?
  - `context_window`: Maximum token capacity.
  - `token_budget`: Maximum safe token limit allocated for single turns.
- **`model_factory.py`**:
  - Dynamically constructs the correct Pydantic AI model client based on the requested capability key (e.g. `fast_reasoning`, `deep_thought`, `code_generation`).
  - Translates requests to OpenAI-compatible endpoints or native socket connections.
- **`conflicts.py` & VRAM Arbitration**:
  - Manages GPU resource contention. If an agent requests a heavy 70B model while another service is using VRAM, LychD handles orderly shutdown, cache flushing, and reactivation.
- **`secret_isolation.py`**:
  - Ensures API keys and secret tokens are injected directly into the adapter memory and never logged, leaked to the database, or echoed back in agent responses.
