---
description: Produce a telecoms sector landscape report anchored in ANTA's methodology and live Supabase state — operator universe state, ANTA dimensions read, AI adoption patterns, vendor ecosystem, regulatory, themes, commissioning-ready angles
argument-hint: "[scope, optional, e.g. 'EMEA tier-1'] [cycle, optional, default most recent] [theme angle, optional]"
---

# Sector Overview (Telco) Command

Build the editorial-and-methodology landscape report for the telecoms sector. Anchored in ANTA scoring data + extraction-table content for the chosen cycle.

## Workflow

### Step 1: Parse the inputs

- **Scope** — optional; default full ANTA universe. Common alternatives: "EMEA tier-1", "APAC", "vendor ecosystem only", "operator universe only".
- **Cycle** — optional; default most recent completed `scoring_cycles` row. Specify `scoring_cycle_id` UUID or cycle name.
- **Depth** — optional; `summary` / `standard` (default) / `deep`.
- **Audience** — optional; default `analyst-editorial`. Other values: `cxo`, `vendor-strategy`, `regulatory`, `anta-steering-board`.
- **Theme angle** — optional; if specified, the overview leans into that theme rather than neutral landscape.

### Step 2: Verify and load

1. Pull the operator + vendor universe in scope from `anta-supabase`.
2. Pull `cycle_snapshots` for the anchor cycle + prior cycle for movement analysis.
3. Pull `scores` for the anchor cycle, aggregated by dimension + indicator.
4. Pull all extraction-table rows (`ai_mentions`, `ai_capex`, `vendor_mentions`, `earnings_call_themes`) linked to source_docs in the cycle window.
5. Pull `v_composite_scores` and `v_rankings` for top/bottom-of-list ranking.

### Step 3: Invoke the skill

Use `skill: "sector-overview-telco"` to assemble the landscape report.

### Step 4: Deliver output

Provide:

1. **Markdown report** with the fixed structure: executive summary / operator universe state / ANTA dimensions read / AI adoption patterns / vendor ecosystem / regulatory landscape / editorial themes / notable absences / commissioning-ready angles / methodology + caveats.
2. **Word-count check** — confirm length matches the `depth` parameter (summary ~1.5K, standard ~3.5K, deep ~7K).

### Step 5: Offer next steps

After delivering, offer:
- "Want me to expand any of the commissioning-ready angles into full `/tv-angle` drafts?"
- "Want me to drill into any single operator's trajectory via `operator-trajectory-tracker`?"
- "Want me to compare this cycle's read to the prior cycle in detail via `/filing-diff` at universe scale?"
- "Want me to package this for the ANTA steering board with the audience reframed?"

## Quality Checklist

Before delivery, confirm:
- [ ] Cycle anchor confirmed and stated in the report
- [ ] Archetype + trajectory + composite distributions traced to actual `cycle_snapshots` rows
- [ ] All 4 ANTA dimensions covered with universe-wide medians + named extremes
- [ ] AI adoption section grounded in real extracted counts (not vibes)
- [ ] Vendor ecosystem section anchored in actual `vendor_mentions` rows
- [ ] Editorial themes traced to `earnings_call_themes` rows
- [ ] Notable absences specific, not generic
- [ ] Commissioning-ready angles each name a specific filer/pattern + framing
- [ ] Methodology + caveats section present unless `depth = summary`
- [ ] Length matches `depth` parameter
- [ ] No equity-research artefacts (no sector trading multiples, valuation calls, stock-pick framing)
