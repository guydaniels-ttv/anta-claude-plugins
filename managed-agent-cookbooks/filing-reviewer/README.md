# Filing Reviewer — managed-agent template

## Overview

Operator or vendor filing + earnings call → model update → analyst note draft. Same source as the [`filing-reviewer`](../../plugins/agent-plugins/filing-reviewer) Cowork plugin — this directory is the Managed Agent cookbook for `POST /v1/agents`.

## Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
# Once the ANTA Supabase MCP and filings-store MCP are stood up, export their URLs too.
../../scripts/deploy-managed-agent.sh filing-reviewer
```

## Steering events

See [`steering-examples.json`](./steering-examples.json). Fan out across the ANTA coverage universe from your orchestration layer — one session per operator or vendor.

## Security & handoffs

Transcripts, press releases, and filings are untrusted. Three-tier isolation:

| Tier | Touches untrusted docs? | Tools | Connectors |
|---|---|---|---|
| **`transcript-reader`** | **Yes** | `Read`, `Grep` only | None |
| `model-updater` / Orchestrator | No | `Read`, `Grep`, `Glob`, `Agent` | (ANTA Supabase MCP, read-only — once wired) |
| **`note-writer`** (Write-holder) | No | `Read`, `Write`, `Edit` | None |

`transcript-reader` returns length-capped, schema-validated JSON. `note-writer` produces `./out/note-<ticker>.docx` and the updated model at `./out/model-<ticker>.xlsx`.
