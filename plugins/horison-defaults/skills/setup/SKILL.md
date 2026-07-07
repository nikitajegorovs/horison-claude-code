---
name: setup
description: Guide for configuring Horison MCP server API keys. Use when an MCP server fails to connect, when the user asks about setup, or when environment variables are missing.
---

# Horison MCP Setup Guide

## MCP servers that need environment variables

Neo4j and Langfuse are included in the plugin's `.mcp.json` but need env vars to connect. Add these to your shell profile (`~/.zshrc` or `~/.bashrc`):

### Neo4j Cypher MCP (graph databases)

The plugin registers **three** Neo4j MCP servers — one per Horison graph environment:

| Server | Graph (Horison environment) | Env var prefix |
|--------|------------------------------|----------------|
| `neo4j-prod` | Horison Prod — tenant graph, **production** (cabde9ed) | `PROD_NEO4J_*` |
| `neo4j-dev` | Horison Dev — tenant graph, **development** (37d16874) | `DEV_NEO4J_*` |
| `neo4j-ta` | Horison TA — consultancy benchmarking graph | `TA_NEO4J_*` |

All run side-by-side as independent stdio subprocesses and surface as separately-namespaced tools (`mcp__...neo4j-prod__*`, `mcp__...neo4j-dev__*`, `mcp__...neo4j-ta__*`), so you can query any graph in the same session. **`neo4j-prod` is production data — prefer `neo4j-dev` for anything exploratory or write-bearing.**

> Each server is registered with a deliberately **unique `command`+`args`** (`uvx …` / `uvx --quiet …` / `uv tool run …`) because the plugin loader dedupes by argv and ignores `env`. They launch a byte-identical server; the distinct argv is only to register all three. See the `neo4j-mcp` skill for details.

```bash
# Horison Prod (neo4j-prod server)
# For Aura: use neo4j+s:// (encrypted). Find URI in Aura Console → instance → Connect
export PROD_NEO4J_URI="neo4j+s://cabde9ed.databases.neo4j.io"
export PROD_NEO4J_USERNAME="neo4j"
export PROD_NEO4J_PASSWORD="your-prod-password"
export PROD_NEO4J_DATABASE="neo4j"

# Horison Dev (neo4j-dev server)
export DEV_NEO4J_URI="neo4j+s://37d16874.databases.neo4j.io"
export DEV_NEO4J_USERNAME="neo4j"
export DEV_NEO4J_PASSWORD="your-dev-password"
export DEV_NEO4J_DATABASE="neo4j"

# Horison TA (neo4j-ta server)
export TA_NEO4J_URI="neo4j+s://yyyyyyyy.databases.neo4j.io"
export TA_NEO4J_USERNAME="neo4j"
export TA_NEO4J_PASSWORD="your-ta-password"
export TA_NEO4J_DATABASE="neo4j"
```

If you only work on one of the graphs, set just that prefix. Any unconfigured server appears as **failed** in `/mcp` (the Neo4j driver rejects an empty URI at connect time) — that's expected and doesn't affect the working ones.

> **Note:** The plugin pins `fastmcp<3` to avoid a known incompatibility with `mcp-neo4j-cypher`.

### Supabase (application database)

The plugin registers **two** project-scoped Supabase MCP servers; both authenticate with a
single personal access token via a Bearer header (the hosted MCP's OAuth needs org-admin
approval, so we use a PAT):

```bash
# Supabase PAT — dashboard → Account → Access Tokens → Generate new token
export SUPABASE_ACCESS_TOKEN="sbp_..."
```

| Server | Project | Use |
|--------|---------|-----|
| `supabase-dev` | dev/staging `qbdyiyoaleuppddcsaxk` | **writable** — author migrations, schema, RLS |
| `supabase-prod` | prod `nwhtkmaujbrhwjbesixt` | **reads/debug only** — ⚠️ writable, but never author schema (goes via `horison-migrations` → gated `push-prod`) |

Each server's URL `project_ref` scopes which project it reaches. See the **`supabase-mcp`**
skill for the authoring workflow.

### Langfuse (prompt management)

```bash
# Encode your API keys as Base64: echo -n "pk-lf-XXX:sk-lf-XXX" | base64
export LANGFUSE_MCP_AUTH="<base64-encoded-pk:sk>"
```

Get keys from **Langfuse → Project Settings → API Keys**. The plugin prepends `Basic ` automatically.

> **Note:** The MCP endpoint requires Langfuse **v3.125.0+**.

### Horison App MCP (deal / KG tools)

The plugin registers **two** Horison MCP servers. **Both use WorkOS OAuth in the
browser** — no env vars or tokens to set:

| Server | Target | WorkOS env | Setup |
|--------|--------|-----------|-------|
| `horison-prod` | `https://mcp.horison.ai/mcp` (hardcoded) | Production | None — `/mcp → Authenticate` |
| `horison-dev` | `http://localhost:8010/mcp` (default) | Staging | None — run the server, then `/mcp → Authenticate` |

`horison-prod` works immediately. `horison-dev` targets a server you run locally
(`make mcp` in agentic-chat-service); until it's running it shows as **failed** in
`/mcp` — harmless if you're not doing local MCP development.

```bash
# Optional — repoint horison-dev away from localhost:8010 (must be a registered
# WorkOS resource indicator, e.g. the deployed staging service):
# export DEV_HORISON_MCP_URL="https://staging---horison-mcp-iosxkhzrva-ew.a.run.app/mcp"
```

See the **`horison-mcp`** skill for the full run-locally and add-a-tool workflow
(including the `localhost:3000` consent frontend needed for the local OAuth flow).

## MCP servers included in the plugin (no setup needed)

| Server | Auth |
|--------|------|
| **Langfuse Docs** | No auth required |
| **Context7** | No auth required |
| **Playwright** | No auth required |
| **Memory** | No auth required |
| **Serena** | No auth required |

## If an MCP server fails

- Check env vars are set: `echo $NEO4J_URI`, `echo $TA_NEO4J_URI`, `echo $LANGFUSE_MCP_AUTH`
- For stdio servers (npx/uvx): ensure `node`/`npx` or `uv`/`uvx` is installed
- Check `~/.claude.json` for conflicting entries: `enabledMcpjsonServers: []` blocks all plugin servers, `disabledMcpServers` can explicitly disable servers
- Servers with missing env vars fail silently — they won't appear in `/mcp`
- Langfuse MCP returning 404? Your instance needs v3.125.0+ — check with `curl -s https://your-instance/api/public/health`
- Restart Claude Code after any config changes — MCP servers connect at startup
