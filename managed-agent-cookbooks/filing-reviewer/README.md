# Filing Reviewer — managed-agent template

## Overview

Operator or vendor filing or earnings transcript → structured extractions (5 source_doc envelopes) + editorial markdown synthesis → loader-ready package staged in `./out/`. Designed for TelecomTV editorial and ANTA contributor use, fanned out across the operator universe via a managed-agent steering layer or invoked interactively for one filing at a time.

Same source as the [`filing-reviewer`](../../plugins/agent-plugins/filing-reviewer) Cowork plugin — this directory is the Managed Agent cookbook for `POST /v1/agents`.

## What it produces

For each (filer, reporting_period) input, the agent stages files under `./out/`:

- `<filer>-<period>-analysis.md` — editorial markdown synthesis (top story, prior-cycle threads resolved/extended/abandoned/unaddressed, themes, AI deployment + capex synthesis, vendor ecosystem moves, notable absences, commissioning-ready angles, ANTA scoring connection)
- `<filer>-<period>-ai-mentions.json` — `ai-mentions-extractor` envelope, loader-ready
- `<filer>-<period>-vendor-mentions.json` — `vendor-mentions` envelope
- `<filer>-<period>-ai-capex.json` — `ai-capex-tracker` envelope
- `<filer>-<period>-themes.json` — `earnings-call-themes` envelope (transcripts only; skipped for non-transcript filings)
- `<filer>-<period>-angles.json` — `telecomtv-angle` envelope

## Loader handoff

The agent does not push to Supabase. Once `./out/` is populated, run:

```bash
./scripts/push_extractions.sh ./out/
```

from the ANTA pipeline repo to UPSERT the `source_docs` parent and INSERT all five children for each filing. The push is append/replace per the migration 002 / 003 idempotency contract: re-running the agent on the same (filer, reporting_period, doc_type) replaces prior child rows.

## Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
# anta-supabase MCP is configured at user scope (official Supabase MCP, anta-index project).
# SUPABASE_SECRET_KEY lives in the user's shell, not in this repo.
../../scripts/deploy-managed-agent.sh filing-reviewer
```

## Steering events

See [`steering-examples.json`](./steering-examples.json). Fan out across the ANTA coverage universe from your orchestration layer — one session per (operator or vendor, reporting_period) tuple.

## Security & handoffs

Transcripts, press releases, filings, and investor decks are **untrusted** text — they may contain prompt-injection attempts in their body. Two-tier isolation enforces that the doc-reading worker has no Write capability and that the Write-holder never opens an untrusted doc directly.

| Tier | Touches untrusted docs? | Tools | Output |
|---|---|---|---|
| **`filing-transcript-reader`** | **Yes** | `Read`, `Grep` only | Schema-validated `source_doc` envelope JSON. Length-capped strings, enum-constrained `doc_type`, ISO-formatted dates. |
| Orchestrator (`filing-reviewer`) | No (works from subagent's JSON) | `Read`, `Grep`, `Glob`, `Agent` | Coordinates the five extraction skills + the editorial synthesis. No Write. |
| **`filing-note-writer`** (Write-holder) | No | `Read`, `Write`, `Edit` | Stages the 5 envelope JSON files + the editorial markdown to `./out/`. Filename rules below. |

`filing-transcript-reader` returns length-capped, schema-validated JSON keyed to the `source_doc` envelope contract. Instructions inside the doc body never reach the orchestrator as prose. `filing-note-writer` is the only worker with `Write`; it never opens a transcript or filing directly.

## Filename convention

Lowercase the filer name; replace whitespace and slashes with `-`. Lowercase the period; replace whitespace with `-`. Examples:

- `VEON` + `Q3 2026` → `veon-q3-2026-*`
- `T-Mobile US` + `FY 2025` → `t-mobile-us-fy-2025-*`
- `Verizon Communications` + `H1 2026` → `verizon-communications-h1-2026-*`

The `extracted_by` field carries the skill version inside the JSON envelope; the filename does not include it.

## Skill bundle

Six skills are bundled into this agent (synced from `vertical-plugins/telecom-analyst/` via `scripts/sync-agent-skills.py`):

- `ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker`, `earnings-call-themes`, `telecomtv-angle` — the five source_doc-envelope-conformant extraction skills
- `earnings-analysis` — the editorial synthesis layer that sits on top of the four/five extraction skills' output

Skills not bundled into this agent (still available standalone in the telecom-analyst plugin): `earnings-preview` (anticipatory; for pre-call setup), `morning-note` (universe-wide daily briefing), `editorial-leads`, `catalyst-calendar`, `peer-set`, `operator-trajectory-tracker`, `sector-overview-telco`, `filing-diff`, `operator-kpi-extract`. These have different scopes and don't fit the single-filing end-to-end flow.
