---
name: sector-overview-telco
description: Produce a telecoms sector landscape report framed around ANTA's 4 dimensions × 5 indicators methodology — covers operator universe state (archetypes, trajectories, AI-readiness composite), vendor ecosystem map, regulatory landscape, AI adoption patterns across the industry, and editorial themes worth covering. Optionally scoped to a region or tier. Designed for TelecomTV editorial pitch material, ANTA steering board prep, or industry-research deliverables. Use when the user asks for a sector overview / sector report / industry landscape / state of the telecoms industry / ANTA cycle summary / where the operator universe stands today.
---

# Sector Overview (Telco)

Produce a telecoms-industry landscape report anchored in ANTA's methodology and live Supabase state. The output is editorial / methodology, not equity-research. It synthesises operator-universe state (archetypes, trajectories, composite scores), vendor ecosystem patterns, regulatory developments, and AI adoption — into a deliverable suitable for a TelecomTV pitch, an ANTA steering board readout, or external industry-research consumption.

## When to Use

Use when the user requests:
- "Sector overview of the telecoms industry"
- "Where does the operator universe stand right now?"
- "Industry report for [region]"
- "ANTA cycle summary for the steering board"
- "Telco landscape for the run-up to MWC"
- "What does AI adoption look like across the operator universe this cycle?"

**Do NOT use if:**
- The user wants a single operator's view → use `operator-trajectory-tracker`
- The user wants a forward calendar of events → use `catalyst-calendar`
- The user wants peer-set work for a single subject → use `peer-set`
- The user wants editorial story leads → use `editorial-leads`

## Inputs

| Input | Required | Notes |
|---|---|---|
| Scope | Optional | Default: full ANTA universe (tier-1 + tier-2 operators + tracked vendors). Common alternatives: "EMEA tier-1", "APAC", "vendor ecosystem only", "operator universe only". |
| Cycle | Optional | Default: most recent completed ANTA scoring cycle. Specify e.g. "2026-Calibration-May" to anchor to a different cycle. |
| Depth | Optional | `summary` (~1,500 words) / `standard` (~3,500 words, default) / `deep` (~7,000 words). |
| Audience | Optional | Default `analyst-editorial`. Other values: `cxo`, `vendor-strategy`, `regulatory`, `anta-steering-board`. |
| Theme angle | Optional | If the user has a specific theme to centre the overview around (e.g. "AI-RAN buildout", "GenAI customer-care adoption", "vendor consolidation"), pass it. The overview will lean into that theme rather than producing a neutral landscape. |

## Output

Markdown report with a fixed structure. Length scales with `depth` parameter.

```
# Telecoms Sector Overview — [Cycle name] — [Date]

**Scope:** [region / universe scope]
**Cycle anchor:** [scoring cycle name + date]
**Theme angle:** [if applicable, otherwise "neutral landscape"]

## Executive summary
[3–6 sentences. The single most important read on the state of the universe right now.]

## Operator universe state

**Archetype distribution**
[Count + share of universe in each archetype, anchored to `cycle_snapshots` for the relevant cycle. Note any meaningful shifts vs. the prior cycle.]

**Trajectory distribution**
[Count + share by trajectory state, anchored to `cycle_snapshots`. Note movers vs. prior cycle.]

**Composite-score distribution**
[Histogram-style summary: top quartile, median, bottom quartile. Name the operators at each end.]

**Notable movers**
[3–6 operators that shifted archetype or trajectory between cycles. One sentence each on why.]

## ANTA dimensions read

For each of the 4 dimensions:

**[Dimension name]**
- Universe-wide median score on this dimension this cycle
- Indicator-level highlights (which indicators drive the variance)
- Operator examples at the top + bottom of the dimension
- 1–2 sentences on the editorial story this dimension is telling this cycle

## AI adoption patterns

[Synthesised from `ai_mentions` + `ai_capex` rows across the universe in this cycle.

- Total AI mentions count (universe-wide); category mix (strategy / deployment / contribution / hype / risk distribution)
- Quantified AI capex disclosed across the universe; spent vs. committed vs. announced ratios
- Most-mentioned AI vendors across the universe (cross-references `vendor_mentions`)
- Operators with the most concrete in-production AI deployments
- Operators with high AI-strategy-to-deployment talk ratios (i.e. lots of strategy, little execution)]

## Vendor ecosystem

[Synthesised from `vendor_mentions` rows across the universe in this cycle.

- Top hyperscaler partnerships across the operator universe
- Top NEPs by mention count + materiality
- Notable new vendor relationships this cycle (vendor_universe_candidates that have been promoted)
- Vendors that have appeared in fewer mentions than the prior cycle (potential ecosystem shift)]

## Regulatory landscape

[Region-relevant regulatory developments this cycle: spectrum, M&A approvals, AI Act / sectoral AI rules, data protection. 2–4 paragraphs.]

## Editorial themes

[3–6 themes identified across the universe this cycle, drawn from `earnings_call_themes` rows. Each:
- Theme title + category
- How widespread (count of operators where it surfaced)
- Where the editorial tension is (operators that have it as a tension theme)
- Suggested commissioning angle if any]

## Notable absences

[Industry-level absences: themes you'd expect across the universe given recent industry movement that aren't showing up in disclosure. 2–4 bullets.]

## Commissioning-ready angles for TelecomTV

[3–6 short bullets, each pointing at a piece an editor could commission. Cross-reference back to specific evidence above.]

## Methodology + caveats

[1 paragraph. Cycle anchor, data freshness, any data-quality issues that affected the read. Especially important if any operator's `cycle_snapshots` row was missing or stale.]
```

For `depth = summary`, collapse Operator universe state + ANTA dimensions read + AI adoption patterns into a single ~600-word section. Drop Methodology + caveats.

For `depth = deep`, expand each section: per-indicator breakdowns under ANTA dimensions, regional sub-tables under operator universe state, full per-vendor tables under vendor ecosystem.

## Workflow

### Step 1: Determine scope + cycle

1. Confirm scope (default: full universe).
2. Confirm cycle anchor (default: most recent completed `scoring_cycles` row).
3. Confirm depth + audience + optional theme angle.

### Step 2: Pull the universe state

From `anta-supabase`:
- `operators` rows in scope
- `cycle_snapshots` rows for the anchor cycle (gives archetype, trajectory, composite_score, DoD, disclosure_quality per operator)
- Prior cycle's `cycle_snapshots` rows for movement analysis

If many operators have missing `cycle_snapshots` for the anchor cycle, surface this in the methodology + caveats section.

### Step 3: Pull the ANTA dimensions read

From `anta-supabase`:
- `scores` rows for the anchor cycle, aggregated by `dimension` (4) and `indicator` (20)
- `v_composite_scores` and `v_rankings` views for top/bottom-of-list ranking

For each dimension: median + top/bottom 5 operators + most variance-contributing indicators.

### Step 4: Pull AI adoption + vendor ecosystem patterns

- `ai_mentions` rows from `source_docs` linked to operators in scope, dated within the cycle window. Aggregate by `category` and by named vendor.
- `ai_capex` rows linked to operators in scope, dated within the cycle window. Aggregate by `disclosure_status` and `ai_scope`.
- `vendor_mentions` rows linked to operators in scope. Aggregate by `vendor_canonical` and `vendor_type`.

These are the evidence base for the AI adoption + vendor ecosystem sections.

### Step 5: Pull editorial themes + tensions

- `earnings_call_themes` rows linked to operators in scope, dated within the cycle window.
- Aggregate by `category`. Identify themes that recur across multiple operators.
- For each cross-universe theme, count operators with `tension = TRUE` on that theme — high count = industry-wide tension.

### Step 6: Pull regulatory developments

Web search + `news_items` query for regulatory developments dated within the cycle window, scoped to the regions of operators in scope. Don't over-dramatise — focus on developments that have material implications for multiple operators.

### Step 7: Identify notable absences at industry level

Cross-check against:
- Themes from prior cycles that should have recurred but didn't
- Industry-level commitments (e.g. operator coalitions, joint ventures) that should have matured
- Major peer movements that should have prompted reactions across the universe

### Step 8: Write commissioning-ready angles

3–6 bullets. Each names a specific filer or specific cross-universe pattern. Each cross-references the supporting evidence already in the report.

### Step 9: Write methodology + caveats

Cycle anchor, data freshness, any operators with missing scoring data, any data-quality issues.

## Quality checklist

Before delivery:
- [ ] Cycle anchor confirmed and stated
- [ ] Archetype + trajectory + composite distributions traced to `cycle_snapshots`
- [ ] All 4 dimensions covered with universe-wide medians + named extremes
- [ ] AI adoption section grounded in actual extracted counts (not vibes)
- [ ] Vendor ecosystem section anchored in real `vendor_mentions` rows
- [ ] Editorial themes traced to `earnings_call_themes` rows; cross-universe themes identified
- [ ] Notable absences specific, not generic
- [ ] Commissioning-ready angles each name a specific filer or pattern + concrete framing
- [ ] Methodology + caveats section present (unless `depth = summary`)
- [ ] Length matches `depth` parameter
- [ ] No equity-research artefacts (no sector trading multiples, no valuation calls, no investment-implications-as-stock-picks framing)

## Important notes

- This skill leans heavily on `anta-supabase` MCP and the live ANTA scoring data. Without it, the operator-universe-state and ANTA-dimensions sections are unworkable. If MCP is unavailable, downscope to a vendor + regulatory + recent-themes report and flag the limitation.
- The `connection to ANTA scoring` framing is genuine here — this skill is the place where editorial output meets methodology output. Be explicit about which claims are derived from ANTA scores and which are from `ai_mentions` / `vendor_mentions` extractions.
- For audience = `anta-steering-board`, lean into the methodology layer: dimension scores, archetype shifts, indicator-level commentary. For audience = `analyst-editorial`, lean into the themes + commissioning angles. The difference is in section weighting, not section presence.
- For `theme angle` (e.g. "AI-RAN buildout"): all sections should be tilted toward that theme rather than neutral. Vendor ecosystem section becomes "AI-RAN vendor ecosystem", AI adoption section becomes "AI-RAN deployment patterns", etc.

## Resources

### ../../references/source-doc-envelope.md
The shared `source_doc` envelope convention used across the telecom-analyst extraction skills (relevant context for the AI adoption and vendor ecosystem sections, both of which synthesise from envelope-shaped extractions).

## Dependencies

**Required:**
- `anta-supabase` MCP — for the operator universe, `cycle_snapshots`, `scores`, all extraction tables, `v_composite_scores`, `v_rankings`

**Optional:**
- `news_items` MCP / web search — for regulatory developments
- `peer-set` skill output — useful for the "notable movers" section if the report wants to highlight peer-relative shifts
- `telecomtv-archive` MCP — to flag commissioning-ready angles that TelecomTV has already covered (avoid duplicate recommendations)
