# Architectural Decision Record (ADR) 001: Tech Stack Selection

**Date:** 2026-02-03
**Status:** Accepted

## Context
The project requires a cross-platform (Windows/Linux) personal assistant with:
1.  **Privacy:** Local-first data and encryption.
2.  **Extensibility:** Ability to add new skills/agents dynamically.
3.  **Modern DX:** Strong typing and ecosystem support for rapid MVP development.
4.  **Backend Capabilities:** Vector search, Scheduling, Auth, and Realtime updates.

## Decision
We have decided to use the following stack:

1.  **Language:** **TypeScript (Node.js)**
    *   *Why:* Unified language for CLI and Backend. Excellent ecosystem for AI (LangChain.js equivalents, Vercel AI SDK patterns) and CLI tools (Commander/Oclif). Strong typing ensures maintainability.
2.  **Infrastructure:** **Self-Hosted Supabase (Docker)**
    *   *Why:* Provides a "Backend-in-a-Box" replacing manual setup of SQLite + Vector Extensions + Cron + Auth.
    *   *Features Used:* `Postgres` (Data), `pgvector` (Memory), `pg_cron` (Scheduling), `Supabase Vault` (Secrets), `Realtime` (Updates).
3.  **Communication:** **HTTP/REST (Fastify)**
    *   *Why:* Simple, standard, debuggable. Fastify provides high performance and low overhead compared to Express.
4.  **CLI Framework:** **Commander.js**
    *   *Why:* Industry standard for Node.js CLIs. Easy to build subcommands and help menus.

## Consequences
*   **Pros:** Significantly faster development speed (Auth, Vector, DB are pre-integrated). TypeScript provides safety. Docker ensures consistent deployment across OS.
*   **Cons:** Higher resource footprint than a compiled binary (Rust/Go) + SQLite. Requires Docker to be installed on the user's machine.

---

# Architectural Decision Record (ADR) 002: Multi-Agent Architecture

**Date:** 2026-02-03
**Status:** Accepted

## Context
The assistant needs to handle complex, vague user requests ("Plan my trip") that require multiple distinct skills (Research, Calendar, Booking).

## Decision
We will implement a **Hierarchical Supervisor-Worker Pattern**.
1.  **Supervisor:** A "Router" agent that analyzes the initial intent.
2.  **Workers:** Specialized agents (e.g., `SchedulerAgent`, `ResearchAgent`) that possess specific tools.
3.  **Policy Layer:** A lightweight routing model (Machine Learning Classifier) that predicts the best worker for a task based on input embeddings.

## Consequences
*   **Pros:** Modular. New skills can be added as new Workers without breaking the core logic.
*   **Cons:** Latency increased by the "Routing" step. Complexity in managing inter-agent state.

---

# Architectural Decision Record (ADR) 003: Memory Management

**Date:** 2026-02-03
**Status:** Accepted

## Context
The assistant needs to remember user preferences (Long-term) and the current conversation context (Short-term).

## Decision
We will use a **Tiered Memory System**:
1.  **Short-Term:** Stored in a Postgres `conversations` table (structured JSON). Pruned/Summarized after N turns.
2.  **Long-Term:** Stored in `pgvector`.
    *   *Process:* A nightly cron job summarizes Short-Term memory and embeds it into the Long-Term vector store.

## Consequences
*   **Pros:** efficient retrieval. Keeps the context window clean for LLMs.
*   **Cons:** "Nightly" consolidation means immediate learning might be delayed until the next sync (acceptable for MVP).
