---
description: Extract standard telco operator KPIs from a filing or results pack and produce normalised snapshots (one per segment) ready for the ANTA cycle_snapshots loader
argument-hint: "[path or URL to source doc] [operator name, optional] [cycle, optional]"
---

# Operator KPI Extract Command

Scan a single primary-source operator document and produce normalised snapshots — one per (operator, segment, cycle) tuple — matching ANTA's canonical KPI vocabulary.

## Workflow

### Step 1: Parse the inputs

- **Source doc** — path or URL to a results pack, transcript, 10-K, 20-F, half-year, full-year, quarterly, or investor day
- **Operator name** — optional; infer from the doc if not given
- **Cycle** — optional; infer from the doc using the operator's fiscal-year convention

If the source doc is missing, ask:
- "Which document should I scan? Paste a path or URL."

### Step 2: Verify and load

1. Fetch the doc via `filings-store` MCP if a URL.
2. Verify the doc is the latest cycle in the operator's fiscal convention.
3. Look up the operator in `anta-supabase` — confirm canonical name. **Reject if the filer is a vendor**; this skill is operator-scoped.
4. Query `anta-supabase` for the operator's prior `cycle_snapshots` row(s) to get the canonical segment list.

### Step 3: Invoke the skill

Use `skill: "operator-kpi-extract"` to do the extraction and normalisation.

### Step 4: Deliver output

Provide three things:

1. **Markdown summary tables — one per segment** — with the canonical KPIs in rows and {value, unit, yoy_pct, yoy_organic_pct, source_loc, confidence} in columns. Mark `kpis_not_disclosed` clearly below each table.

2. **JSON object** matching the schema in the skill's `SKILL.md`, ready for the `cycle_snapshots` loader to consume.

3. **3–6 sentence narrative** for a TelecomTV editor / ANTA analyst. Lead with headline group financials. Note any restatements and any consistency-check failures.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to feed this into the ANTA `cycle_snapshots` loader?"
- "Want me to also run `/ai-mentions` and `/ai-capex` on the same doc?"
- "Want me to run `/filing-diff --focus guidance` against the prior cycle?"
- "Want me to flag any new segments for review against the operator universe?"

## Quality Checklist

Before delivery, confirm:
- [ ] Source doc verified as latest cycle in the operator's fiscal convention
- [ ] Operator matched to ANTA canonical name; filer_side confirmed as operator
- [ ] All disclosed segments captured (group + every reported region/country/business unit)
- [ ] Every canonical KPI searched per segment; absent ones recorded in `kpis_not_disclosed` (not silently dropped)
- [ ] Every disclosed KPI traces to a `source_loc`
- [ ] Units explicit: monthly vs annualised churn, EBITDA vs EBITDAaL, organic vs reported YoY
- [ ] Currency captured per segment — never silently converted
- [ ] Derived KPIs flagged `calculated: true`
- [ ] Restatements captured with the reason quoted
- [ ] Consistency checks run; failures noted in the summary
- [ ] No fabricated values
- [ ] Output JSON validates against the schema
