---
name: operator-kpi-extract
description: Extract standard telco operator KPIs (revenue, EBITDA, EBITDAaL, EBITDA margin, FCF, capex, net debt, leverage; mobile subs / postpaid / prepaid / 5G; fixed broadband / FTTH; mobile ARPU / fixed ARPU; mobile postpaid churn / fixed churn; 5G pops covered, fibre passings, take-up rate; headcount) from an operator's filing or earnings call. Produce one normalised snapshot per (operator, segment, cycle) tuple, matching ANTA's canonical KPI naming, ready for the cycle_snapshots loader. Use when the user asks to extract KPIs, key metrics, headline numbers, subscriber numbers, ARPU, churn, fibre stats, 5G stats, capex intensity, or financial highlights from an operator's filing, transcript, 10-K, 20-F, quarterly, half-year, full-year results, or investor day. Operator-scoped — for vendor filings use a vendor-KPI skill instead.
---

# Operator KPI Extract

Extract the standard telco operator KPI set from a single source document and produce normalised snapshots — one per (operator, segment, cycle) tuple — ready to land in ANTA's `cycle_snapshots` table.

**The honest premise:** operators report KPIs inconsistently — different definitions (EBITDA vs EBITDAaL), different units (mobile subs in millions vs thousands), different currencies (group reporting vs local-currency segments), different segments (geographic / customer / product), and frequent restatements. This skill normalises all of that against a canonical KPI vocabulary, captures what is and isn't disclosed, and is honest about ambiguity rather than coercing numbers to fit a clean schema.

## When to Use

Use when the user requests:
- "Extract KPIs from [operator]'s H1 results"
- "Pull the headline numbers / key metrics from this 20-F"
- "What are [operator]'s ARPU and churn this quarter?"
- "5G subs and fibre passings from this annual report"
- "Build a cycle_snapshot row for [operator] from this filing"
- "Headline financials for [operator] Q3"

**Do NOT use if:**
- The filer is a vendor / equipment maker (Ericsson, Nokia, Amdocs, NVIDIA) → vendor KPI set is different; use a vendor-KPI skill when it exists, or extract bespoke
- The user wants AI-specific extraction → use `ai-mentions-extractor` or `ai-capex-tracker`
- The doc lacks a financial / operating disclosure (e.g. a press release announcing a single deal) → the skill needs an actual KPI-bearing doc

## Inputs

| Input | Required | Notes |
|---|---|---|
| Source document | Yes | Filing PDF/HTML, results presentation, transcript, results press release, investor deck. Path or URL. |
| Operator name | If not derivable | Cross-check against ANTA operator universe. |
| Cycle | If not derivable | E.g. Q3 FY26, H1 FY26, FY 2025. Use the operator's own fiscal-year convention. |
| Reporting currency | Optional | Pulled from the doc. Used to flag mismatches across segments. |

## Output

JSON object with a `snapshots` array (one per segment), plus a markdown summary table per segment, plus a 3–6 sentence narrative.

```json
{
  "operator": "Vodafone Group",
  "cycle": "H1 FY26",
  "source_doc": "Vodafone H1 FY26 Results Presentation",
  "fiscal_period_end": "2026-09-30",
  "reporting_currency": "EUR",
  "snapshots": [
    {
      "segment": "group",
      "segment_type": "group | region | country | business-unit | product",
      "currency": "EUR",
      "kpis": {
        "revenue_total": { "value": 18500, "unit": "millions", "raw": "€18.5bn", "yoy_pct": 1.2, "yoy_organic_pct": 2.4, "source_loc": "p. 3 results pack", "confidence": "high" },
        "revenue_service": { "value": 16200, "unit": "millions", "raw": "€16.2bn", "yoy_pct": 0.9, "yoy_organic_pct": 2.1, "source_loc": "p. 3", "confidence": "high" },
        "ebitda_aL": { "value": 6000, "unit": "millions", "raw": "€6.0bn", "yoy_pct": 3.5, "yoy_organic_pct": 4.8, "source_loc": "p. 3", "confidence": "high" },
        "ebitda_margin_pct": { "value": 32.4, "unit": "pct", "raw": "32.4%", "yoy_delta_bps": 70, "source_loc": "p. 3", "confidence": "high" },
        "capex_total": { "value": 3200, "unit": "millions", "raw": "€3.2bn", "yoy_pct": -1.0, "source_loc": "p. 8", "confidence": "high" },
        "capex_intensity_pct": { "value": 17.3, "unit": "pct", "raw": "17.3%", "calculated": true, "source_loc": "calc from capex/revenue", "confidence": "medium" },
        "mobile_subs_total": { "value": null, "raw": null, "source_loc": null, "confidence": null }
      },
      "kpis_not_disclosed": ["mobile_subs_total", "mobile_postpaid_churn_pct", "fixed_arpu_local", "tower_count"],
      "notes": "Group consolidated; excludes Italy disposal. EBITDAaL is the disclosed format — EBITDA before leases is not given."
    },
    {
      "segment": "Germany",
      "segment_type": "country",
      "currency": "EUR",
      "kpis": {
        "revenue_total": { "value": 6400, "unit": "millions", "raw": "€6.4bn", "yoy_pct": -1.8, "yoy_organic_pct": null, "source_loc": "p. 12", "confidence": "high" },
        "mobile_subs_total": { "value": 30.2, "unit": "millions", "raw": "30.2m", "yoy_pct": 0.5, "source_loc": "p. 12", "confidence": "high" },
        "mobile_postpaid_churn_pct": { "value": 1.1, "unit": "pct_monthly", "raw": "1.1% monthly", "yoy_delta_bps": null, "source_loc": "p. 12", "confidence": "high" },
        "ftth_passings": { "value": 4.2, "unit": "millions", "raw": "4.2m homes passed", "yoy_pct": null, "source_loc": "p. 12", "confidence": "medium" }
      },
      "kpis_not_disclosed": ["mobile_arpu_blended", "fixed_arpu_local"],
      "notes": "Segment KPIs are in EUR (Germany is reporting currency)."
    }
  ],
  "restatements": [
    {
      "prior_cycle": "FY25",
      "kpi": "revenue_total (group)",
      "prior_value": 36100,
      "current_restated_value": 35900,
      "reason_quote": "Restated for the disposal of Italy operations.",
      "source_loc": "p. 22"
    }
  ],
  "consistency_checks": [
    {
      "check": "Sum of segment revenue_total vs group revenue_total",
      "result": "match",
      "tolerance_pct": 1.0,
      "note": "Within ±1% — passes."
    },
    {
      "check": "Disclosed EBITDA margin vs calculated EBITDA / revenue",
      "result": "match",
      "note": "Disclosed 32.4%, calculated 32.4%."
    }
  ],
  "summary": "Group revenue €18.5bn (+1.2% reported, +2.4% organic). EBITDAaL €6.0bn at 32.4% margin (+70bps). Capex €3.2bn (17.3% of revenue). Germany softer at -1.8% reported revenue but mobile subs +0.5%. No churn or ARPU disclosed at group level — segment-only. One FY25 restatement for the Italy disposal."
}
```

## Canonical KPI Vocabulary

The skill produces KPI values keyed by ANTA's canonical names. See [references/canonical-kpis.md](references/canonical-kpis.md) for the full list, units, definitions, and synonyms. Summary:

**Income statement:**
- `revenue_total`, `revenue_service`, `revenue_mobile_service`, `revenue_fixed_service`, `revenue_b2b`, `revenue_b2c`, `revenue_equipment`
- `ebitda`, `ebitda_aL` (EBITDA after leases), `ebitda_margin_pct`, `ebitda_aL_margin_pct`
- `net_income`

**Cash flow / balance sheet:**
- `free_cash_flow`, `equity_free_cash_flow`, `operating_free_cash_flow`
- `capex_total`, `capex_intensity_pct`, `capex_excl_spectrum`, `spectrum_capex`
- `net_debt`, `leverage_x` (net debt / EBITDA(aL))

**Subscribers:**
- `mobile_subs_total`, `mobile_subs_postpaid`, `mobile_subs_prepaid`, `mobile_subs_5g`
- `fixed_broadband_subs`, `ftth_subs`, `tv_subs`
- `convergent_customers`
- `mobile_net_adds_postpaid`, `fixed_broadband_net_adds`

**ARPU (always specify currency in unit):**
- `mobile_arpu_postpaid`, `mobile_arpu_prepaid`, `mobile_arpu_blended`
- `fixed_arpu`, `convergent_arpu`

**Churn:**
- `mobile_postpaid_churn_pct`, `mobile_prepaid_churn_pct`, `fixed_broadband_churn_pct` (specify monthly vs annualised in `unit`)

**Network footprint:**
- `pops_covered_5g`, `pops_covered_5g_sa`
- `ftth_passings`, `ftth_take_up_pct`
- `tower_count`
- `spectrum_holdings_mhz`

**Operational:**
- `headcount`

**Use canonical keys verbatim.** If the operator discloses a KPI you can't map cleanly to canonical, capture it in a `custom_kpis` sub-object under that segment, with the raw label and a short explanation in `notes`.

## Workflow

### Step 1: Identify and verify

1. Confirm doc is the latest cycle (filing date verified).
2. Identify the operator; cross-check against ANTA operator universe; reject if `filer_side: vendor`.
3. Identify the cycle in the operator's fiscal convention (Vodafone H1 FY26 is calendar H1 of fiscal year ending Mar 2026, etc.).
4. Note the reporting currency.

### Step 2: Identify segments

Look up the operator's prior cycle_snapshot in Supabase for the canonical segment list. Then scan the doc for:

- Group consolidated → always one snapshot, `segment: "group"`
- Geographic segments → one per disclosed country / region
- Business segments → if the operator reports B2B / B2C / Enterprise separately
- Product segments → mobile / fixed / TV if disclosed independently

If a new segment is named that isn't in the operator universe (e.g. an operator has rebranded or restructured), flag in `universe_candidates`.

### Step 3: Extract each segment's KPIs

For each segment, work through the canonical KPI list. For each KPI:
1. Search the doc for the canonical name and known synonyms (see references/canonical-kpis.md)
2. If found: capture `value`, `unit`, `raw` (as displayed), `yoy_pct` and/or `yoy_organic_pct` if given, `yoy_delta_bps` for margins/percentages, `source_loc` (page or timestamp), `confidence`
3. If not found: add to `kpis_not_disclosed` for that segment

**Don't fabricate values.** If a KPI is given for "operations" or "B2C" but not for the segment in question, leave it absent and add to `kpis_not_disclosed`.

### Step 4: Compute derived KPIs

A few derived KPIs the skill calculates where the inputs are present:
- `capex_intensity_pct` = `capex_total` / `revenue_total` × 100 — calculate when both are disclosed; flag `calculated: true`
- `ebitda_margin_pct` / `ebitda_aL_margin_pct` — calculate when EBITDA and revenue are disclosed; flag `calculated: true`
- `leverage_x` = `net_debt` / `ebitda` (or `ebitda_aL`) — calculate when both are disclosed and the operator does not otherwise disclose leverage; flag `calculated: true`

Always prefer the **disclosed** value over the calculated one when both are present.

### Step 5: Restatements

Scan for explicit restatement language:
- "restated for," "as restated," "previously reported," "comparative period restated," "prior period adjusted for"

For each restated KPI, populate `restatements` with the prior cycle, KPI name, old value, new restated value, and the reason quoted.

### Step 6: Consistency checks

Run these checks and populate `consistency_checks`:

1. **Segment-to-group totals.** Sum of segment revenue_total, EBITDA, capex against group totals — within ±1% is `match`, outside is `mismatch`.
2. **Disclosed vs calculated ratios.** Where the operator discloses EBITDA margin and you can calculate EBITDA/revenue from the same disclosure, check they match.
3. **YoY arithmetic.** Where current and prior period absolute values are both given and YoY% is disclosed, verify the YoY% is consistent.
4. **Net adds = ΔSubs.** Where `mobile_net_adds_postpaid` is disclosed alongside opening and closing `mobile_subs_postpaid`, check arithmetic.

Each check goes in the output. **Failures don't block delivery** — they're flagged for human review, often signalling a real disclosure issue.

### Step 7: Summary

3–6 sentence narrative for a TelecomTV editor / ANTA analyst. Lead with the most editorially significant move. Include:
- Headline group financials (revenue, EBITDA, margin)
- Most material segment delta
- Any restatements
- Any consistency-check failures worth flagging

## Output Schema

See the example above. Required top-level fields: `operator`, `cycle`, `source_doc`, `fiscal_period_end`, `reporting_currency`, `snapshots[]`, `restatements[]`, `consistency_checks[]`, `summary`. Each KPI value object has shape:

```json
{
  "value": "number or null",
  "unit": "string — see canonical-kpis.md for the unit of each canonical KPI",
  "raw": "string — as displayed in the doc, e.g. '€18.5bn' or '1.1% monthly'",
  "yoy_pct": "number or null — reported YoY",
  "yoy_organic_pct": "number or null — organic / underlying YoY if disclosed separately",
  "yoy_delta_bps": "number or null — for percentage/margin KPIs",
  "calculated": "boolean — true if derived, omit or false otherwise",
  "source_loc": "string — page number or transcript timestamp",
  "confidence": "high | medium | low | null"
}
```

## Quality Checklist

Before delivery:
- [ ] Source doc verified as latest cycle in the operator's fiscal convention
- [ ] Operator matched to ANTA canonical name; rejected if `filer_side: vendor`
- [ ] All disclosed segments captured (not just group)
- [ ] Every canonical KPI searched per segment; absent ones recorded in `kpis_not_disclosed`
- [ ] No fabricated values — every disclosed KPI traces to a `source_loc`
- [ ] Units captured explicitly: monthly vs annualised churn, EBITDA vs EBITDAaL, organic vs reported YoY
- [ ] Currency captured per segment (group EUR with local-currency subsidiaries is common)
- [ ] Derived KPIs flagged `calculated: true`
- [ ] Restatements captured with the reason quoted
- [ ] Consistency checks run; any failures noted in the summary
- [ ] If new segments appear that aren't in the operator universe, surfaced explicitly
- [ ] Output JSON validates against the schema

## Resources

### references/canonical-kpis.md
The full canonical KPI list — name, definition, unit, synonyms, ANTA convention, and worked examples for the awkward ones (EBITDA vs EBITDAaL, organic vs reported, monthly vs annualised churn).

## Dependencies

**Required:**
- `WebFetch` (URLs) / `Read` (local paths) — to fetch source documents
- `anta-supabase` MCP — to look up canonical operator name, canonical segment list from prior `cycle_snapshots`, and to confirm `filer_side: operator`

**Optional:**
- `ai-mentions-extractor` skill — run alongside when the doc is a results pack to capture both the KPIs and the AI narrative in one session
- `filing-diff` skill — to surface KPIs that have moved materially cycle-over-cycle once snapshots are landed in Supabase
