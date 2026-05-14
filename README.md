# ANTA Claude Plugins — Telecoms Analyst

Claude Code / Cowork plugins for a **Telecoms Analyst** persona, focused on operator and vendor financial filings (annuals + quarterlies) and earnings calls. Optimised for extracting AI strategy, AI deployment, and AI contribution evidence, alongside the wider telco facts ANTA and TelecomTV track.

Forked from [`anthropics/financial-services`](https://github.com/anthropics/financial-services) and adapted: the equity-research vertical → `telecom-analyst`, the earnings-reviewer agent → `filing-reviewer`. All other FSI verticals, agents, partner plugins, and the Microsoft 365 installer were removed.

Available **two ways from one source**: install it as a [Claude Cowork](https://claude.com/product/cowork) plugin, or deploy it through the [Claude Managed Agents API](https://docs.claude.com/en/api/managed-agents) behind your own workflow engine. Same system prompt, same skills — you choose where it runs.

> [!IMPORTANT]
> Nothing here is investment, legal, tax, or accounting advice. The agent drafts analyst work product — notes, model updates, summaries — for editorial review. Every output is staged for human sign-off. ANTA contributors are responsible for verifying outputs.

## What's in the repo

- **`plugins/agent-plugins/filing-reviewer/`** — the end-to-end agent. Operator or vendor filing/call → KPI + AI-mention extraction → model update → draft note. Self-contained (bundles its own skills).
- **`plugins/vertical-plugins/telecom-analyst/`** — the underlying skills and slash commands. Install on its own if you just want `/earnings`, `/ai-mentions`, `/peers`, `/sector` etc. without the full agent.
- **`plugins/vertical-plugins/financial-analysis/`** — shared modelling/Excel skills and the **MCP connector config** (`.mcp.json`). The `telecom-analyst` plugin depends on this.
- **`managed-agent-cookbooks/filing-reviewer/`** — `agent.yaml` + leaf-worker subagents for headless deployment via `/v1/agents`.

## Getting Started

### Cowork

Settings → Plugins → Add plugin → paste this repo URL `https://github.com/guydaniels-ttv/anta-claude-plugins`, then pick the plugins you want from the marketplace list.

### Claude Code

```bash
# Add the marketplace
claude plugin marketplace add guydaniels-ttv/anta-claude-plugins

# Core (install first — carries the MCP connectors)
claude plugin install financial-analysis@anta-claude-plugins

# Telecom-analyst skills + commands
claude plugin install telecom-analyst@anta-claude-plugins

# Full agent (optional — bundles its own copy of the relevant skills)
claude plugin install filing-reviewer@anta-claude-plugins
```

### Claude Managed Agents

```bash
export ANTHROPIC_API_KEY=sk-ant-...
scripts/deploy-managed-agent.sh filing-reviewer
```

The deploy script resolves file references, uploads skills, creates leaf-worker subagents, and POSTs the orchestrator to `/v1/agents`. See [`scripts/orchestrate.py`](./scripts/orchestrate.py) for a reference event loop.

## How It Fits Together

| | What it is | Where it lives |
|---|---|---|
| **Agent** | Self-contained plugin that owns the end-to-end filing-review workflow — system prompt plus the skills it uses. | `plugins/agent-plugins/filing-reviewer/` |
| **Skills** | Domain expertise, conventions, and step-by-step methods Claude draws on automatically when relevant. Authored once in the vertical; the agent bundles a synced copy. | `plugins/vertical-plugins/telecom-analyst/skills/` (source) · `plugins/agent-plugins/filing-reviewer/skills/` (bundled) |
| **Commands** | Slash actions you trigger explicitly (`/earnings`, `/ai-mentions`, …). | `plugins/vertical-plugins/telecom-analyst/commands/` |
| **Connectors** | [MCP servers](https://modelcontextprotocol.io/) that wire Claude to ANTA's data — Supabase, filings store, TelecomTV archive. | `plugins/vertical-plugins/financial-analysis/.mcp.json` |
| **Managed-agent wrapper** | `agent.yaml` + depth-1 subagents for headless deployment. | `managed-agent-cookbooks/filing-reviewer/` |

Everything is file-based — markdown and JSON, no build step.

## Connectors

`.mcp.json` currently holds **stubs** for the three connectors we plan to wire:

| Connector | Purpose |
|---|---|
| `anta-supabase` | MCP server over the ANTA Supabase (operators, vendors, telco universe, cycle_snapshots). The agent checks what's already known before re-reading a filing. |
| `filings-store` | MCP server over the filings folder (PDFs of annual reports, 10-Ks/20-Fs, quarterlies, transcripts). |
| `telecomtv-archive` | Optional: TelecomTV editorial archive, so the agent can cite prior coverage alongside primary filings. |

Replace the placeholder URLs in [`.mcp.json`](./plugins/vertical-plugins/financial-analysis/.mcp.json) once each MCP server is stood up.

## Making It Yours

These are starting templates — they get better with ANTA-specific tuning.

- **Swap connectors** — edit `.mcp.json` (above).
- **Add ANTA context** — drop ANTA's frameworks, taxonomies, and TelecomTV editorial conventions into skill files.
- **Adjust agent scope** — edit `plugins/agent-plugins/filing-reviewer/agents/filing-reviewer.md` (the system prompt) to match how an ANTA analyst actually runs the workflow.
- **Add your own skills** — copy the structure under `plugins/vertical-plugins/telecom-analyst/skills/` for new extraction or analysis routines.

## Skill & Command Reference

Inherited from upstream `equity-research`, kept as a scaffold to be rewritten with telco vocabulary:

| Skill | Command | Description (to be rewritten for telco) |
|---|---|---|
| earnings-analysis | `/earnings` | Post-quarter operator/vendor update |
| earnings-preview | `/earnings-preview` | Pre-quarter scenario + key metrics |
| initiating-coverage | `/initiate` | Long-form initiation on an operator or vendor |
| model-update | `/model-update` | Roll the model with new actuals |
| morning-note | `/morning-note` | TelecomTV-style daily note draft |
| sector-overview | `/sector` | Telecom sub-sector / theme landscape |
| thesis-tracker | `/thesis` | Maintain evolving thesis on AI strategy & deployment |
| catalyst-calendar | `/catalysts` | Upcoming catalysts (results, MWC, spectrum auctions, etc.) |
| idea-generation | `/screen` | Filter the ANTA universe by criteria |

Skills planned but not yet authored — see the project plan:

- `ai-mentions-extractor` (`/ai-mentions`)
- `ai-capex-tracker` (`/ai-capex`)
- `vendor-mentions` (`/vendors`)
- `operator-kpi-extract` (`/kpis`)
- `filing-diff` (`/filing-diff`)
- `earnings-call-themes` (`/call-themes`)
- `telecomtv-angle` (`/tv-angle`)
- `peer-set` (`/peers`)

## Contributing

Fork the branch, edit, PR. For new content:

- New skill → add it under `plugins/vertical-plugins/telecom-analyst/skills/`, then run `python3 scripts/sync-agent-skills.py` to propagate to the agent.
- Run `python3 scripts/check.py` before pushing — it lints every manifest and verifies cross-file references.

## License

[Apache License 2.0](./LICENSE) (inherited from upstream).
