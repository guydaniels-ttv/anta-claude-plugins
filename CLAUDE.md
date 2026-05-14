# ANTA Claude Plugins — Telecoms Analyst

Cowork plugins and Claude Managed Agent templates for a telecoms-analyst persona, focused on operator and vendor financial filings (annuals + quarterlies) and earnings calls. Optimised for extracting AI strategy / deployment / contribution evidence and broader telco facts.

Forked from `anthropics/financial-services`. Only three plugins are kept: `financial-analysis` (core), `telecom-analyst` (vertical), and `filing-reviewer` (agent).

## Repository Structure

```
├── plugins/
│   ├── agent-plugins/
│   │   └── filing-reviewer/
│   │       ├── .claude-plugin/plugin.json
│   │       ├── agents/filing-reviewer.md   # ← canonical system prompt
│   │       └── skills/                     # ← bundled copies, synced from vertical-plugins/
│   └── vertical-plugins/
│       ├── financial-analysis/             # core modelling/Excel skills + .mcp.json
│       └── telecom-analyst/                # telco skills + commands
├── managed-agent-cookbooks/
│   └── filing-reviewer/
│       ├── agent.yaml                      # system + skills → ../../plugins/agent-plugins/filing-reviewer/...
│       ├── subagents/*.yaml                # depth-1 leaf workers
│       ├── steering-examples.json
│       └── README.md
└── scripts/                                # deploy-managed-agent.sh, check.py, validate.py, orchestrate.py, sync-agent-skills.py
```

Run `python3 scripts/check.py` before committing — it lints every manifest, verifies all `system.file` / `skills.path` / `callable_agents.manifest` references resolve, and fails if any `agent-plugins/filing-reviewer/skills/` copy has drifted from its `vertical-plugins/telecom-analyst/skills/` source. **Edit skills in `vertical-plugins/telecom-analyst/`**, then run `python3 scripts/sync-agent-skills.py` to propagate into the agent bundle.

## Key Files

- `.claude-plugin/marketplace.json`: marketplace manifest — registers the three plugins
- `<plugin>/.claude-plugin/plugin.json`: per-plugin metadata
- `vertical-plugins/telecom-analyst/commands/*.md`: slash commands (`/earnings`, `/ai-mentions`, …)
- `vertical-plugins/telecom-analyst/skills/*/SKILL.md`: detailed extraction and analysis routines
- `vertical-plugins/financial-analysis/.mcp.json`: MCP connector stubs (anta-supabase, filings-store, telecomtv-archive — to be wired)

## Development Workflow

1. Edit markdown files directly — changes take effect immediately
2. Test commands with `/plugin:command-name` syntax
3. Skills are invoked automatically when their trigger conditions match
