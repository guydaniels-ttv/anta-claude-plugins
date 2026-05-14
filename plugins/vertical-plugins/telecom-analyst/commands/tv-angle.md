---
description: Draft TelecomTV editorial angles from a single primary-source document — hook, evidence, novelty check, recommended format, and risks per angle
argument-hint: "[path or URL to source doc] [filer name, optional] [reporting period, optional] [audience hint, optional]"
---

# TelecomTV Angle Command

Read a single primary-source document and produce 2–5 editorial angle drafts an editor could commission against. Each angle is anchored to verbatim evidence and honestly graded on novelty.

## Workflow

### Step 1: Parse the inputs

- **Source doc** — path or URL to a filing PDF/HTML, transcript, investor deck, or press release
- **Filer name** — optional; infer from the doc if not given. Determines `filer_kind` (operator vs. vendor) and `filer_operator_id` via `anta-supabase` lookup.
- **Reporting period** — optional; infer from the doc if not given (e.g. "Q3 2026"). Goes into `source_doc.reporting_period`.
- **Doc type** — optional; infer from the doc if not given. Required for the loader UPSERT.
- **ANTA scoring cycle** — optional; pass `scoring_cycle_id` if the run is tied to an ANTA cycle.
- **Audience hint** — optional; default `analyst-editorial`. Other values: `cxo`, `vendor-strategy`, `regulatory`.

If the source doc is missing, ask:
- "Which document should I draft angles from? Paste a path or URL."

### Step 2: Verify and load

1. Fetch the doc via `filings-store` MCP if a URL.
2. Confirm it is the latest cycle.
3. Look up the filer in `anta-supabase` — canonical name + `filer_operator_id`.
4. Check whether structured-data extractions for this doc already exist (run earlier in this session or in `anta-supabase`). If yes, use them; don't re-extract.
5. Pull peer-set context for this filer from `anta-supabase` — used for `competitive_context`.
6. If `telecomtv-archive` MCP is available, prepare to query it during the freshness check; if not, set expectation that `novelty` will be `"unknown"`.

### Step 3: Invoke the skill

Use `skill: "telecomtv-angle"` to do the angle drafting, evidence anchoring, freshness check, and risks framing.

### Step 4: Deliver output

Provide three things:

1. **Markdown summary table** sorted by `confidence` desc then `novelty` (fresh > incremental > restatement > unknown):

   | # | Headline draft | Type | Audience | Novelty | Length | Format | Confidence |
   |---|---|---|---|---|---|---|---|

2. **JSON object** matching the schema in the skill's `SKILL.md` — `source_doc` envelope + `angles` array + `summary` — ready for Supabase ingest.

3. **3–5 sentence narrative** for the commissioning editor. Lead with the strongest angle. Note which to commission now and which to wait on.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to ingest these angles into the ANTA Supabase `telecomtv_angles` table?" (Pending table design — flag if not yet available.)
- "Want me to draft the lead paragraph for the strongest angle?"
- "Want me to run `/themes`, `/ai-mentions`, `/vendors`, `/ai-capex` on the same doc to deepen the evidence layer?"
- "Want me to run `/peer-set` to widen the competitive context?"

## Quality Checklist

Before delivery, confirm:
- [ ] Filer matched to ANTA canonical name; `filer_kind` correctly set; `filer_operator_id` populated for operator filers
- [ ] 2–5 angles drafted
- [ ] Every angle has verbatim quote(s) in `supporting_evidence`
- [ ] `novelty` honestly assessed — `"unknown"` is acceptable if the archive MCP isn't available, and must be stated explicitly
- [ ] `freshness_basis` populated for every angle
- [ ] `competitive_context` either genuine (from Supabase) or omitted — never fabricated
- [ ] `headline_draft` is specific, not clickbait
- [ ] `risks` lists specific failure modes, not generic disclaimers
- [ ] Output JSON validates against the schema
