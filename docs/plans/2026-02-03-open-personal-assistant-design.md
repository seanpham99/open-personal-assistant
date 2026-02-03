# Open Personal Assistant Design Document

**Date:** 2026-02-03  
**Status:** Draft  
**Project:** Open Personal Assistant

## 1. Executive Summary
A privacy-first, cross-platform personal assistant designed for Windows and Linux. It features a CLI-first interface, a multi-agent swarm for specialized tasks, and a self-evolving architecture that improves performance and skill routing through feedback loops and automated maintenance.

## 2. System Architecture

### 2.1 Overview
The system employs a **Daemon-Client** model:
- **`assistant-d` (Daemon)**: A background process running a FastAPI server. It manages the agent swarm, cron engine, encrypted database, and memory.
- **`assistant` (CLI)**: A lightweight command-line interface for user interaction.

### 2.2 Agent Swarm (Multi-Agent System)
- **Supervisor-Worker Pattern**: A central Supervisor agent receives user intent, decomposes it, and routes tasks to specialized workers (e.g., Scheduler, Note-Taker, Researcher).
- **Dynamic Skills**: Agents can discover, install, and prune skills (tools) from `skills.sh`. 
- **Self-Evolution**: A RAG-based routing policy (lightweight classifier) that learns from user feedback and execution metrics to improve task delegation.

### 2.3 Memory System
- **Short-Term Memory**: Conversation context and transient task states stored in a KV table within SQLite.
- **Long-Term Memory**: User preferences, historical task data, and learned behaviors stored as vector embeddings using `sqlite-vec`.

## 3. Data & Security

### 3.1 Persistence
- **Primary Store**: SQLite database encrypted via SQLCipher.
- **Secure Vault**: Encrypted storage for 3rd-party API keys (Fernet encryption).

### 3.2 IPC & Security
- **Protocol**: HTTP/1.1 (JSON) over Localhost.
- **Authentication**: API Key-based (`X-API-Key`) with keys stored in the user's secure configuration directory.

## 4. Automation & Maintenance (Cron Engine)
An internal async scheduler handles:
- **System Health**: Cache invalidation and data synchronization.
- **Evolution**: Nightly retraining of the routing policy and summary of short-term memory into long-term vectors.
- **Skill Lifecycle**: Automated pruning of low-usage or poor-performing skills.

## 5. Technology Stack
- **Language**: Python 3.12+ (managed by `uv`).
- **Web/API**: FastAPI, `httpx`.
- **CLI**: Typer.
- **Database**: SQLite, `sqlite-vec`, `sqlcipher3`.
- **ML/Evolution**: Scikit-learn (Policy Layer), Sentence-Transformers (Embeddings).
- **Validation/Tooling**: Pydantic, Ruff, Mypy.
