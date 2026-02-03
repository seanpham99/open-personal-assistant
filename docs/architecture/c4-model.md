# C4 Architecture Model: Open Personal Assistant

## 1. System Context (Level 1)

**Description:** A privacy-first, cross-platform personal assistant that uses AI agents to manage tasks, schedule events, and perform research, while keeping user data secure and local.

```mermaid
C4Context
  title System Context Diagram for Open Personal Assistant

  Person(user, "User", "Interacts via Telegram to manage tasks and agents.")
  System(assistant, "Open Personal Assistant", "Manages tasks, runs agents, and encrypts data.")
  
  System_Ext(telegram, "Telegram API", "Messaging Platform for User Interface.")
  System_Ext(supabase, "Supabase (Self-Hosted)", "Provides Auth, Database, Vector Store, and Realtime sync.")
  System_Ext(skills_sh, "skills.sh", "External marketplace/registry for discovering new Agent skills.")
  System_Ext(external_apis, "3rd Party APIs", "Google, GitHub, Linear, etc. accessed by Agents.")

  Rel(user, telegram, "Sends Messages", "HTTPS")
  Rel(telegram, assistant, "Webhooks/Updates", "HTTPS")
  Rel(assistant, supabase, "Persists state & Vectors", "Postgres Protocol/HTTP")
  Rel(assistant, skills_sh, "Downloads Skills", "HTTPS")
  Rel(assistant, external_apis, "Performs Actions", "HTTPS")
```

## 2. Container Diagram (Level 2)

**Description:** The high-level deployable units of the architecture.

```mermaid
C4Container
  title Container Diagram

  Person(user, "User", "Telegram User")

  Container_Boundary(pa_system, "Personal Assistant System") {
    Container(bot_service, "Telegram Bot Service", "Node.js/Telegraf", "Handles message parsing, command routing, and user state.")
    Container(daemon, "Assistant Daemon", "Node.js/Fastify", "Core logic engine. Manages agents, scheduling, and API/Tool execution.")
    Container(policy_engine, "Policy & Routing Engine", "Python/ONNX or JS", "Lightweight model to decide which Agent/Tool to use based on intent.")
  }

  Container_Boundary(infra, "Infrastructure (Docker)") {
    ContainerDb(postgres, "PostgreSQL", "Supabase", "Stores tasks, logs, agent configurations.")
    Container(pgvector, "pgvector", "PostgreSQL Ext", "Stores embeddings for long-term memory.")
    Container(vault, "Supabase Vault", "PostgreSQL Ext", "Encrypts and manages 3rd-party API keys.")
    Container(realtime, "Realtime Server", "Elixir/Phoenix", "Push notifications for Bot updates.")
  }

  Rel(user, bot_service, "Messages", "Telegram Protocol")
  Rel(bot_service, daemon, "RPC/HTTP", "Localhost:3000")
  Rel(daemon, postgres, "Reads/Writes", "Postgres Driver")
  Rel(daemon, vault, "Decrypts Keys", "SQL")
  Rel(daemon, policy_engine, "Inference", "Internal Call")
  Rel(daemon, realtime, "Publishes Events", "WebSocket")
```

## 3. Component Diagram (Level 3) - Daemon

**Description:** Internal components of the `Assistant Daemon`.

```mermaid
C4Component
  title Component Diagram - Assistant Daemon

  Container(daemon, "Assistant Daemon", "Node.js/Fastify")

  Component(api_server, "API/Webhook Server", "Fastify", "Receives webhooks from Telegram Bot Service.")
  Component(supervisor, "Supervisor Agent", "TypeScript Class", "Orchestrates user intent. Decomposes tasks.")
  Component(skill_mgr, "Skill Manager", "Service", "Discovers, installs, and sandboxes skills from skills.sh.")
  Component(memory_mgr, "Memory Manager", "Service", "Handles Short-term (Cache) and Long-term (Vector) retrieval.")
  Component(cron_worker, "Cron Worker", "Node-Schedule / pg_cron", "Executes maintenance and scheduled user tasks.")
  
  Rel(api_server, supervisor, "Delegates Request")
  Rel(supervisor, skill_mgr, "Request Tool Exec")
  Rel(supervisor, memory_mgr, "Fetch Context")
  Rel(cron_worker, supervisor, "Triggers Scheduled Agents")
```

## 4. Code Diagram (Level 4) - Agent Structure

**Description:** Implementation details of the Agent class.

```mermaid
classDiagram
  class Agent {
    +string id
    +string persona
    +Tool[] skills
    +execute(Task task) Result
    +learn(Feedback feedback) void
  }

  class SupervisorAgent {
    +decompose(string intent) SubTask[]
    +route(SubTask task) Agent
  }

  class Tool {
    +string name
    +Schema inputSchema
    +run(Context ctx) output
  }

  Agent <|-- SupervisorAgent
  Agent o-- Tool
```