# Open Personal Assistant Design Document

**Date:** 2026-02-03  
**Status:** Draft  
**Project:** Open Personal Assistant

## 1. Executive Summary

A privacy-first, cross-platform personal assistant designed for Windows and Linux. It features a CLI-first interface, a multi-agent swarm for specialized tasks, and a self-evolving architecture that improves performance and skill routing through feedback loops and automated maintenance. This new iteration transitions to a TypeScript-based stack powered by a self-hosted Supabase instance to leverage its robust ecosystem (Auth, Realtime, Vector Store, Cron).

## 2. System Architecture

### 2.1 Overview

The system employs a **Daemon-Client** model built with TypeScript/Node.js:

- **`assistant-d` (Daemon)**: A background process (Node.js/Fastify) acting as the brain. It interfaces with self-hosted Supabase for state, auth, and memory.
- **`assistant` (CLI)**: A lightweight command-line interface (built with `commander` or `oclif`) that communicates with the daemon via local HTTP.

### 2.2 Agent Swarm (Multi-Agent System)

- **Supervisor-Worker Pattern**: A central Supervisor agent receives user intent, decomposes it, and routes tasks to specialized workers.
- **Dynamic Skills**: Agents can discover and integrate tools dynamically.
- **Self-Evolution**:
  - **Policy Layer**: A lightweight routing model (potentially running on Edge Functions or local ONNX) predicts the best agent for a task.
  - **Feedback Loop**: User interactions update the routing policy via Supabase Edge Functions / Database Webhooks.

### 2.3 Supabase-Powered Infrastructure

Instead of a single SQLite file, we leverage the self-hosted Supabase stack:

- **PostgreSQL**: Primary relational store (Tasks, Logs, structured data).
- **pgvector**: Vector store for Long-Term Memory (Embeddings).
- **pg_cron**: Scheduling engine for maintenance and recurring tasks.
- **pg_net / Edge Functions**: For executing skills and external API calls securely.
- **Realtime**: For instant updates to the CLI (e.g., streaming agent thoughts).
- **Vault**: Supabase Vault for secure storage of API keys (encryption at rest).

## 3. Data & Security

### 3.1 Persistence & Memory

- **Short-Term Memory**: Stored in Postgres (or Redis if needed for high throughput) for active context.
- **Long-Term Memory**: `pgvector` stores embeddings of history, preferences, and facts.

### 3.2 Security

- **Authentication**: Supabase Auth (GoTrue) handles user identity. The CLI authenticates once and stores a session token.
- **Secrets**: 3rd-party API keys are stored in **Supabase Vault**, ensuring they are encrypted at rest and only exposed to authorized Edge Functions/Agents.

## 4. Automation & Maintenance

- **System Health**: `pg_cron` jobs handle cache invalidation and database maintenance (VACUUM, etc.).
- **Evolution**: Scheduled jobs analyze `skill_usage` tables to flag underperforming skills or retrain the routing policy.

## 5. Technology Stack

- **Language**: TypeScript (Node.js 20+).
- **Runtime Manager**: `npm` / `pnpm` / `bun` (TBD - sticking to `npm` for standard compatibility).
- **Backend/Daemon**: Fastify (or Hono) + Supabase JS Client.
- **CLI**: Commander.js or Oclif.
- **Database**: Self-Hosted Supabase (Postgres, GoTrue, Realtime, Storage, Vector, Vault).
- **Validation**: Zod.
- **Linting/Quality**: Biome (fast, modern).
