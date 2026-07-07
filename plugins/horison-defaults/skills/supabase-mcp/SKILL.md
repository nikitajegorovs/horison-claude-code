---
name: supabase-mcp
description: How to use the Supabase MCP server tools effectively. Activated when working with Supabase, writing RLS policies, querying the database, managing auth, or interacting with Supabase Storage.
---

# Using the Supabase MCP Server

Claude reaches Supabase through **two** scoped MCP servers (both authenticate with your `SUPABASE_ACCESS_TOKEN` PAT via a Bearer header — no per-session OAuth):

| Server | Project | Use it for |
|---|---|---|
| **`supabase-dev`** | dev/staging `qbdyiyoaleuppddcsaxk` | authoring — `apply_migration`, `execute_sql`, all schema work |
| **`supabase-prod`** | prod `nwhtkmaujbrhwjbesixt` | **reads/debug only** — inspect schema/data, `list_migrations` |

**Author on `supabase-dev`, read on `supabase-prod`.** `supabase-prod` is writable but not the place to author — the only writer of prod schema is `horison-migrations`' gated `push-prod`. The loop:

1. Author on `supabase-dev` via `apply_migration` (never raw `execute_sql` for DDL — it leaves no ledger row, so capture can't see it).
2. `make capture` the ledger row into a `horison-migrations` file → PR → merge → gated `push-prod` applies it to prod.
3. Reads on `supabase-prod` are always fine.

The capture→PR half runs in the `horison-migrations` repo (`make status` / `capture` / `check`, PAT-only) — do it yourself or hand it to your coding agent; the targets are identical either way. Whoever runs it, eyeball the generated SQL before the PR: correct final shape, additive-only (expand-contract), no stray drift.

> ⚠️ **Staging is shared — scope your capture.** The ledger can hold other devs' in-flight rows, and a bare `make capture` files *all* of them, sweeping another feature's migrations into your PR. `apply_migration` returns the exact version of each row you authored, so capture only those: `make capture ONLY=<v1,v2>` (or `SINCE=<last-repo-version>`). A bare `make capture` is fine only when you're the sole author in flight. Either way, `make status` first and confirm every written file is yours before the PR.

Direct prod writes are break-glass only — a reviewed ledger/schema repair that can't go through the pipeline. See `horison-migrations`' `CLAUDE.md`.

## Available Tool Patterns

Tools are prefixed per server (`mcp__…supabase-dev__*` / `mcp__…supabase-prod__*`) — pick by intent.

### Query the database
```
Use the Supabase MCP to run SQL:
  SELECT * FROM documents WHERE status = 'processed' LIMIT 10;
```

### Inspect schema
```
List all tables:
  SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';

Show columns for a table:
  SELECT column_name, data_type, is_nullable FROM information_schema.columns
  WHERE table_name = 'documents';
```

### Check RLS policies
```
SELECT schemaname, tablename, policyname, permissive, roles, cmd, qual, with_check
FROM pg_policies WHERE tablename = 'documents';
```

### Test RLS as a specific role
```sql
-- Test as authenticated user
SET role authenticated;
SET request.jwt.claims = '{"sub": "user-uuid-here", "role": "authenticated"}';
SELECT * FROM documents;  -- Should only return user's docs
RESET role;
```

### Manage storage buckets
Use MCP tools to list buckets, check bucket policies, and manage files.

### Auth operations
Use MCP tools to list users, check auth config, and inspect auth hooks.

## Best Practices

- **Always check RLS before writing queries** — understand what the current user can see
- **Use EXPLAIN ANALYZE** on queries before recommending them for production
- **Prefer parameterized patterns** — avoid string interpolation in SQL
- **Check indexes** before adding new queries: `SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'your_table';`
- **Use transactions** for multi-step operations: `BEGIN; ... COMMIT;`

## Common Horison Patterns

- Documents are stored in Supabase Storage, metadata in `documents` table
- User auth via Supabase Auth (email + OAuth providers)
- RLS enforces per-user access — always verify policies when changing schemas
- Edge Functions handle webhook processing and async tasks
