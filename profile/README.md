# Doki Stack

**AI-driven infrastructure automation with mandatory human approval.**

Doki Stack is an open-core platform where platform engineers describe infrastructure changes in natural language. The system generates, validates, and applies those changes (Terraform, Ansible) with mandatory human-in-the-loop approval at every step.

## Key Principles

- **HITL is mandatory** — No infrastructure change without explicit human approval
- **Fail closed** — If policy or context is unavailable, block
- **Multi-tenant** — `org_id` scoped isolation everywhere
- **Agents learn** — Memory MCP stores preferences and outcomes per org
- **Extensible** — Register custom MCP servers to integrate any data source

## Architecture

```mermaid
graph TB
    subgraph Users
        Browser[Platform Engineer]
    end

    subgraph Frontend["Frontend (TypeScript)"]
        UI[platform-ui]
    end

    subgraph API["API Layer (Go)"]
        API_SRV[api-server]
    end

    subgraph MCP_Rust["MCP Servers (Rust)"]
        SCAN[mcp-scanner]
        EXEC[mcp-execution]
    end

    subgraph MCP_Go["MCP Servers (Go)"]
        POL[mcp-policy]
    end

    subgraph Agents["AI Agents (Python / LangGraph)"]
        ORCH[agent-orchestrator]
        AUTO[agent-automation]
        REV[agent-review]
    end

    subgraph Shared["Shared Libraries"]
        SG[shared-go]
        SR[shared-rust]
        SP[shared-python]
        ST[shared-ts]
        DB[db-schemas]
    end

    subgraph Data["Data & Messaging"]
        PG[(PostgreSQL)]
        MINIO[(MinIO)]
        QDRANT[(Qdrant)]
        DRAGON[(Dragonfly)]
        RMQ[RabbitMQ]
    end

    subgraph Infra["Deployment"]
        K8S[infrastructure]
    end

    Browser --> UI
    UI --> API_SRV
    API_SRV --> ORCH
    API_SRV -->|SSE events| RMQ

    ORCH --> AUTO
    ORCH --> REV
    ORCH --> SCAN
    ORCH --> POL
    ORCH --> EXEC
    AUTO --> SCAN
    AUTO --> POL
    AUTO --> EXEC
    REV --> POL

    SCAN --> PG
    SCAN --> MINIO
    SCAN --> DRAGON
    SCAN --> RMQ
    EXEC --> PG
    EXEC --> MINIO
    POL --> PG
    POL --> QDRANT
    POL --> DRAGON
    API_SRV --> PG
    API_SRV --> DRAGON
    ORCH --> PG

    SR -.->|imported by| SCAN
    SR -.->|imported by| EXEC
    SG -.->|imported by| API_SRV
    SG -.->|imported by| POL
    SP -.->|imported by| ORCH
    SP -.->|imported by| AUTO
    SP -.->|imported by| REV
    ST -.->|imported by| UI
    DB -.->|migrations| PG

    K8S -.->|deploys all| SCAN
    K8S -.->|deploys all| EXEC
    K8S -.->|deploys all| POL
    K8S -.->|deploys all| API_SRV
    K8S -.->|deploys all| UI
    K8S -.->|deploys all| ORCH
    K8S -.->|deploys all| AUTO
    K8S -.->|deploys all| REV

    style Frontend fill:#3178c6,color:#fff
    style API fill:#00ADD8,color:#fff
    style MCP_Rust fill:#dea584,color:#000
    style MCP_Go fill:#00ADD8,color:#fff
    style Agents fill:#3776ab,color:#fff
    style Shared fill:#6c757d,color:#fff
    style Data fill:#336791,color:#fff
    style Infra fill:#326ce5,color:#fff
```

## Repositories

### Community Edition (Apache 2.0)

| Repo | Language | Purpose |
|------|----------|---------|
| [shared-go](https://github.com/Doki-Stack/shared-go) | Go | Shared utilities, logging, OTel |
| [shared-rust](https://github.com/Doki-Stack/shared-rust) | Rust | Shared crate, tracing, OTel |
| [shared-python](https://github.com/Doki-Stack/shared-python) | Python | Shared Pydantic models, OTel |
| [shared-ts](https://github.com/Doki-Stack/shared-ts) | TypeScript | Shared API types for the UI |
| [db-schemas](https://github.com/Doki-Stack/db-schemas) | SQL | Language-agnostic database migrations |
| [mcp-scanner](https://github.com/Doki-Stack/mcp-scanner) | Rust | Repository Scanner MCP |
| [mcp-execution](https://github.com/Doki-Stack/mcp-execution) | Rust | Terraform/Ansible Execution MCP |
| [mcp-policy](https://github.com/Doki-Stack/mcp-policy) | Go | Policy enforcement MCP |
| [mcp-sdk](https://github.com/Doki-Stack/mcp-sdk) | TS + Python | SDK for building custom MCP servers |
| [agent-orchestrator](https://github.com/Doki-Stack/agent-orchestrator) | Python | LangGraph Orchestrator |
| [agent-automation](https://github.com/Doki-Stack/agent-automation) | Python | Automation Agent |
| [agent-review](https://github.com/Doki-Stack/agent-review) | Python | Review Agent |
| [platform-ui](https://github.com/Doki-Stack/platform-ui) | TypeScript | Next.js Platform UI |
| [api-server](https://github.com/Doki-Stack/api-server) | Go | Backend-for-Frontend API |
| [infrastructure](https://github.com/Doki-Stack/infrastructure) | YAML | Kubernetes manifests, GitOps |
| [docs](https://github.com/Doki-Stack/docs) | MkDocs | Documentation |
| [example-mcps](https://github.com/Doki-Stack/example-mcps) | TS + Python | Reference MCP implementations |

## License

Community Edition is licensed under Apache 2.0. Enterprise Edition uses BSL 1.1.
