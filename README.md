# Horison Claude Code Plugin Marketplace

A curated plugin marketplace for [Claude Code](https://claude.com/claude-code) built around the Horison PE deal intelligence stack — GCP, Supabase, Neo4j Aura, and Langfuse.

One install gives you **13 MCP servers**, **10 specialized agents**, **11 skills**, and **20 optional plugin packs** covering Python, TypeScript, infrastructure, data engineering, and more.

## Quick Start

### 1. Add the marketplace and install

```bash
/plugin marketplace add MartinNguyen03/horison-claude-code
/plugin install horison-defaults@horison-claude-code
```

This gives you 13 MCP servers, 10 agents, and 11 skills. Servers that need no env-var setup (Playwright, Context7, Memory, Serena, Langfuse Docs, and both **horison-prod** / **horison-dev**) work out of the box — the Horison servers authenticate via browser OAuth. **Supabase** (a `SUPABASE_ACCESS_TOKEN` PAT), **Neo4j**, and **Langfuse** need environment variables — see below.

### 2. Set environment variables

Add these to your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
# Supabase PAT (dashboard → Account → Access Tokens) — authenticates both supabase-dev and supabase-prod.
export SUPABASE_ACCESS_TOKEN="sbp_..."

# Neo4j — find URI in Aura Console → instance → Connect
export NEO4J_URI="neo4j+s://xxxxxxxx.databases.neo4j.io"
export NEO4J_USERNAME="neo4j"
export NEO4J_PASSWORD="your-password"
export NEO4J_DATABASE="neo4j"

# Langfuse — get keys from Langfuse → Project Settings → API Keys
# echo -n "pk-lf-XXX:sk-lf-XXX" | base64
export LANGFUSE_MCP_AUTH="<base64-encoded-pk:sk>"
```

> **Note:** The plugin prepends `Basic ` automatically. The MCP endpoint requires Langfuse **v3.125.0+**.

### Horison App MCP

Both `horison-prod` (`https://mcp.horison.ai/mcp`) and `horison-dev` (default
`http://localhost:8010/mcp`) authenticate via **WorkOS OAuth in the browser** —
**no env vars or tokens to set.** Run `/mcp`, select the server, and Authenticate.

`horison-dev` targets a server you run locally (`make mcp` in agentic-chat-service);
until it's running it shows as *failed* in `/mcp` — harmless. To repoint it (must
be a registered WorkOS resource indicator):

```bash
# export DEV_HORISON_MCP_URL="https://staging---horison-mcp-iosxkhzrva-ew.a.run.app/mcp"
```

See the **`horison-mcp`** skill for the full run-locally and add-a-tool workflow.

### 3. Restart Claude Code

```bash
source ~/.zshrc  # reload env vars
```

Restart Claude Code and run `/mcp` to verify all servers are connected. Servers with missing env vars will fail silently.

## What's in `horison-defaults`

### MCP Servers (13)

| Server | Type | Auth | Purpose |
|--------|------|------|---------|
| **Supabase Dev** | HTTP | PAT (`SUPABASE_ACCESS_TOKEN`) | **Writable** — author against dev/staging (`qbdyiyoaleuppddcsaxk`): migrations, schema, RLS |
| **Supabase Prod** | HTTP | PAT (`SUPABASE_ACCESS_TOKEN`) | Reads/debug against prod (`nwhtkmaujbrhwjbesixt`) — writable, but schema is authored via `horison-migrations` |
| **Horison Prod** | HTTP | OAuth (browser) | 17 read-only deal / KG / vault tools against production (`mcp.horison.ai`) |
| **Horison Dev** | HTTP | OAuth (browser) | Same tools against a locally-run server (`localhost:8010`) — for tool development |
| **Neo4j ×3** | stdio (`uvx`) | Env vars | Cypher against the `prod` / `dev` / `ta` Neo4j Aura graphs |
| **Langfuse** | HTTP | Env vars | Prompt management, tracing, evaluation |
| **Langfuse Docs** | HTTP | None | Up-to-date Langfuse documentation |
| **Playwright** | stdio (`npx`) | None | Browser automation and testing |
| **Context7** | stdio (`npx`) | None | Up-to-date library documentation |
| **Serena** | stdio (`uvx`) | None | Code-aware AI assistant |
| **Memory** | stdio (`npx`) | None | Persistent key-value memory |

### Agents (10)

| Agent | Model | Focus |
|-------|-------|-------|
| `gcp-architect` | opus | GCP infrastructure, Cloud Run, IAM, networking |
| `supabase-pro` | opus | RLS policies, Edge Functions, Auth, Storage |
| `neo4j-graph-expert` | opus | Cypher, graph modeling, GDS, knowledge graphs |
| `langfuse-ops` | inherit | Tracing, prompt management, evaluation |
| `kg-pipeline-dev` | opus | Document ingestion, entity extraction, KG construction |
| `code-reviewer` | inherit | Code quality, correctness, readability |
| `security-analyzer` | inherit | OWASP Top 10, auth, data handling |
| `performance-analyzer` | inherit | N+1 queries, bottlenecks, complexity |
| `test-writer` | inherit | Unit and integration test generation |
| `api-documenter` | inherit | REST/GraphQL documentation |

### Skills (11)

| Skill | Purpose |
|-------|---------|
| `setup` | API key configuration guide for all MCP servers |
| `horison-mcp` | Connect to, run locally, and add tools to the Horison app MCP |
| `supabase-mcp` | How to use Supabase MCP tools effectively |
| `neo4j-mcp` | Cypher patterns, schema inspection, Aura connection |
| `langfuse-mcp` | Prompt management via Langfuse MCP |
| `horison-architecture` | Platform architecture and data flows |
| `memory` | Tool use policy and post-operation behaviour |

## Troubleshooting

### MCP server won't connect

- Run `/mcp` to see which servers are connected and which failed
- For stdio servers: ensure `node`/`npx` and `uv`/`uvx` are installed
- Check env vars are set: `echo $NEO4J_URI`, `echo $LANGFUSE_MCP_AUTH`
- Servers with missing env vars fail silently — they won't appear in `/mcp`

### `enabledMcpjsonServers: []` blocking plugin servers

If you see this in `~/.claude.json` under your project, it blocks **all** plugin MCP servers. Remove the key entirely or populate it with the servers you want enabled.

### `disabledMcpServers` explicitly disabling servers

Check for entries like `"plugin:horison-defaults:neo4j"` in `disabledMcpServers`. Remove any servers you want active.

### Langfuse MCP returns 404

Your Langfuse instance needs to be **v3.125.0+** for MCP support. Check with `curl -s https://your-instance/api/public/health`.

## Optional Plugins (20)

Install any of these for additional skills and commands:

```bash
/plugin install <plugin-name>@horison-claude-code
```

| Plugin | Category | What it adds |
|--------|----------|-------------|
| `python-development` | Languages | 16 skills, Python 3.12+, FastAPI, async, testing |
| `javascript-typescript` | Languages | JS/TS, advanced types, Node.js patterns |
| `backend-development` | Development | API design, GraphQL, microservices, CQRS |
| `frontend-mobile-development` | Development | Next.js App Router, React, Tailwind, React Native |
| `developer-essentials` | Development | Git, SQL, error handling, auth patterns |
| `llm-application-dev` | AI/ML | LangChain/LangGraph, RAG, vector search, prompts |
| `cloud-infrastructure` | Infra | AWS/Azure/GCP, Terraform, service mesh |
| `kubernetes-operations` | Infra | K8s manifests, Helm, GitOps, security policies |
| `devops` | Infra | CI/CD, GitHub Actions, observability, monitoring |
| `database` | Data | PostgreSQL schema, SQL optimization, migrations |
| `data-engineering` | Data | ETL, Spark, dbt, Airflow, data quality |
| `security` | Security | SAST, threat modeling, OWASP, SOC2/HIPAA/GDPR |
| `testing` | Quality | Unit testing, TDD, red-green-refactor |
| `code-quality` | Quality | Refactoring, tech debt, performance review |
| `debugging` | Quality | Error analysis, trace debugging, distributed systems |
| `documentation` | Docs | OpenAPI, Mermaid diagrams, ADRs, changelogs |
| `git-pr-workflows` | Workflows | Git automation, PR enhancement |
| `conductor` | Workflows | Context-Driven Development workflow |
| `agent-teams` | Workflows | Multi-agent parallel code review and development |
| `framework-migration` | Modernization | Framework upgrades, dependency updates |

## Prerequisites

- **Node.js / npx** — required for most stdio MCP servers
- **uv / uvx** — required for Neo4j and Serena MCP servers (`brew install uv` or `pip install uv`)
- **Claude Code** with plugin marketplace support

## Project Structure

```
horison-claude-code/
├── .claude-plugin/
│   └── marketplace.json       # Plugin catalog
├── plugins/
│   ├── horison-defaults/      # Core plugin (MCP + agents + skills)
│   │   ├── .claude-plugin/plugin.json
│   │   ├── .mcp.json          # 13 MCP server configs
│   │   ├── agents/            # 10 agent definitions
│   │   └── skills/            # 11 skill guides
│   ├── python-development/
│   ├── javascript-typescript/
│   └── ...                    # 18 more optional plugins
└── README.md
```
