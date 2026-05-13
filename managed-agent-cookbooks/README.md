# Managed-agent templates

The `filing-reviewer` agent ships **two ways**: as a Cowork plugin (see [`../plugins/agent-plugins/filing-reviewer/`](../plugins/agent-plugins/filing-reviewer/)) and as a Claude Managed Agent template here. Same agent, same skills — pick your surface. The directory below is a deploy manifest that references the canonical system prompt and skills from the matching plugin, so there is one source of truth.

Run `../scripts/deploy-managed-agent.sh filing-reviewer` to upload skills, create leaf workers, and `POST /v1/agents` with the resolved config.

| Agent | Vertical plugin | Cowork tile | CMA steering event | Leaf workers |
|---|---|---|---|---|
| [`filing-reviewer`](./filing-reviewer/) | telecom-analyst | Operator / vendor filing + call → model update → note draft | `Process filing: <ticker> <period>` | transcript-reader · model-updater · **note-writer** |

**Bold** leaf = the only worker with `Write`.

## Manifest vs API

The `agent.yaml` files use the real `POST /v1/agents` field names with a few conveniences the deploy script resolves:

| Manifest convention | Resolves to |
|---|---|
| `system: {file: ../../plugins/agent-plugins/<slug>/agents/<slug>.md, append: "..."}` | `system: "<inlined contents + append>"` |
| `system: {text: "..."}` | `system: "<text>"` |
| `skills: [{from_plugin: ../../plugins/agent-plugins/<slug>}]` | uploads every `skills/*` under that dir → `[{type: custom, skill_id: ...}, ...]` |
| `skills: [{path: ../../...}]` | `skills: [{type: custom, skill_id: <uploaded-id>}]` |
| `callable_agents: [{manifest: ./subagents/x.yaml}]` | `callable_agents: [{type: agent, id: <created-id>, version: latest}]` |

> **Research preview:** `callable_agents` (multi-agent delegation) supports **one delegation level**. An orchestrator can call workers; workers cannot call further subagents.
