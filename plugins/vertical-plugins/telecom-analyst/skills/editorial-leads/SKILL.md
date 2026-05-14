---
name: editorial-leads
description: Systematic story-lead generation across the ANTA operator and vendor universe. Surfaces editorial leads worth pursuing — operators with high-tension recent themes, fresh AI capex disclosures with peer resonance, vendors making unusual moves, notable industry-wide absences, and pattern-level signals (e.g. multiple operators converging on the same disclosure shape). Designed as the editorial pipeline for TelecomTV — what should we be writing about, ranked by editorial value. Use when the user asks for editorial leads / story ideas / what should we cover / what's worth pursuing / pitch me telecoms stories / what looks interesting across the operator universe / story sweep.
---

# Editorial Leads

Generate ranked editorial story leads across the ANTA operator and vendor universe. The output is a prioritised list of "what TelecomTV should be writing about right now" — anchored in fresh extractions, prior-cycle thread state, and pattern-level signals across multiple filers.

This skill is the **editorial pipeline filler**. Where `morning-note` covers the daily flow and `catalyst-calendar` covers forward events, this skill answers the standing question "what stories are emerging across the universe that we haven't picked up yet?"

## When to Use

Use when the user requests:
- "Editorial leads for the week"
- "What should we be writing about?"
- "Story sweep across the operator universe"
- "Pitch me telecoms stories"
- "What are we missing?"
- "Editorial pipeline check"
- "Find me leads on [theme, e.g. AI-RAN]"
- "What's emerging this cycle that we haven't covered?"

**Do NOT use if:**
- The user wants a single editorial angle from one filing → use `telecomtv-angle`
- The user wants the daily briefing → use `morning-note`
- The user wants forward events → use `catalyst-calendar`
- The user wants industry-wide context (not story leads) → use `sector-overview-telco`

## Inputs

| Input | Required | Notes |
|---|---|---|
| Lookback window | Optional | Default: last 14 days. Common alternatives: "since last cycle", "last 30 days", "since [named event]". |
| Theme filter | Optional | If specified, restricts leads to those touching the theme (e.g. "AI-RAN only", "GenAI customer-care only", "vendor consolidation only"). |
| Universe scope | Optional | Default: full ANTA universe. Can scope to a region, tier, or named subset. |
| Lead count target | Optional | Default: 10. Range 3–25. |
| Audience | Optional | Default `analyst-editorial`. Same vocabulary as `telecomtv-angle`. |

## Output

Markdown ranked leads list with a fixed structure, plus a JSON sidecar.

### Markdown structure

```
# Editorial Leads — [Date]

**Lookback:** [window]
**Theme filter:** [theme or "none — full universe sweep"]
**Universe scope:** [scope]

## Top leads

For each lead in ranked order:

### [N]. [Lead headline] — [editorial-value tier: must-cover / strong-lead / worth-considering]

- **Anchor:** [Filer or pattern (cross-universe). Specific.]
- **Type:** [single-filing | cross-universe pattern | absence-signal | trajectory-shift | vendor-ecosystem-move]
- **What's the story:** [2–3 sentences. The editorial framing. Specific, anchored to evidence.]
- **Supporting evidence:**
  - [evidence pointer 1 — e.g. "VEON Q3 earnings_call_themes row id X — tension theme on Pakistan GenAI CSAT"]
  - [evidence pointer 2 — e.g. "Orange Q3 ai_capex row id Y — comparable GPU spend disclosure"]
- **Novelty:** [fresh | incremental | restatement | unknown] — [one sentence on basis]
- **Suggested format:** [news | feature | analysis | column]
- **Suggested next skill:** [`/tv-angle` for [filer] | `/peer-set` for [filer] | `/filing-diff` for [filer] | none]
- **Why this didn't surface earlier:** [optional — only include if there's a meaningful "why now" answer, e.g. "took until Q3 results ingest for the pattern to be visible across 3 operators"]

---

## Pattern-level signals

[Distinct from individual leads — these are signals that something is moving across the universe but no single filing crystallises it yet. 0–3 entries. Each:
- Pattern description (one sentence)
- Filers contributing to the pattern (named, with evidence pointers)
- Why this is editorially interesting
- Watch trigger — what would need to land to make this a commissioning-ready lead]

## Notable absences

[2–4 bullets. Stories TelecomTV would normally pursue that aren't surfacing in fresh data — could be that the underlying news isn't happening, or could be that we're missing it. Be honest about which.]

## Methodology + caveats

[Lookback boundary, data freshness, any data-quality issues, any operators/vendors with stale extraction data that may have suppressed leads.]
```

### JSON sidecar

```json
{
  "generated_at": "2026-05-14T12:00:00Z",
  "lookback_window": "string",
  "theme_filter": "string or null",
  "universe_scope": "string",
  "leads": [
    {
      "rank": "integer",
      "headline": "string",
      "tier": "must-cover | strong-lead | worth-considering",
      "anchor": {
        "kind": "filer | cross-universe-pattern",
        "filer_name": "string or null",
        "filer_operator_id": "uuid or null",
        "pattern_description": "string or null"
      },
      "type": "single-filing | cross-universe-pattern | absence-signal | trajectory-shift | vendor-ecosystem-move",
      "story": "string — 2 to 3 sentences",
      "supporting_evidence_refs": ["string", ...],
      "novelty": "fresh | incremental | restatement | unknown",
      "novelty_basis": "string — one sentence",
      "suggested_format": "news | feature | analysis | column",
      "suggested_next_skill": "string or null",
      "why_now": "string or null"
    }
  ],
  "pattern_signals": [
    {
      "pattern_description": "string",
      "contributing_filers": ["string", ...],
      "why_interesting": "string",
      "watch_trigger": "string"
    }
  ],
  "notable_absences": ["string", ...]
}
```

## Editorial-value tiers

`must-cover` — Strong editorial signal, fresh, ANTA-resonant, peer-comparable. Should be commissioned this week.

`strong-lead` — Worth commissioning but timing is flexible; may want to wait for adjacent context to land.

`worth-considering` — Genuine signal but lower priority; might be a smaller piece or a contributing fact in a larger one.

Be honest with the distribution. Most leads are `strong-lead` or `worth-considering`. Only the top 1–3 should typically be `must-cover`.

## Lead types

`single-filing` — One filing produced an editorially significant claim that hasn't been covered yet. Most common type.

`cross-universe-pattern` — Multiple filers (≥3) are converging on the same disclosure shape, vendor mention, or theme. The story is the convergence.

`absence-signal` — A filer or set of filers conspicuously *failed* to address something they previously committed to, or that peers are now addressing. Stories where the silence is the news.

`trajectory-shift` — An operator's archetype/trajectory is materially shifting based on recent evidence and warrants editorial coverage of the shift itself.

`vendor-ecosystem-move` — A vendor makes a move (new partnership, scope change, pulled relationship) that has cross-operator implications.

## Workflow

### Step 1: Determine scope

1. Confirm lookback window (default 14 days).
2. Confirm theme filter (default none).
3. Confirm universe scope.
4. Confirm lead count target (default 10).

### Step 2: Pull fresh evidence

From `anta-supabase`, query for everything in the lookback window:
- New `source_docs` rows for in-scope filers
- All child extraction rows (`ai_mentions`, `ai_capex`, `vendor_mentions`, `earnings_call_themes`) attached to those source_docs
- Any `telecomtv_angles` already commissioned/published in the window (to avoid recommending duplicate stories)
- Any `peer_set_runs` from the window (provide peer-context fodder)
- `news_items` rows in the window

### Step 3: Score single-filing leads

For each in-window source_doc:
- High-tension `earnings_call_themes` rows (`tension = TRUE`, `salience = 'high'`) → candidate single-filing leads
- AI capex disclosures with `disclosure_status = 'committed'` or `'spent'` and material size → candidate
- Notable vendor moves (new named-strategic relationship, scope expansion) → candidate
- Quantified AI deployment claims with peer-comparison value → candidate

For each candidate, check if it's already been covered (cross-check `telecomtv_angles` with `review_status = 'commissioned'` or `'published'`). If yes, drop the lead. If not, score it on novelty, peer-comparability, and editorial value.

### Step 4: Detect cross-universe patterns

Look for convergence across 3+ filers in the lookback window:
- Same AI vendor newly named by ≥3 operators → vendor-ecosystem-move pattern
- Same theme category surfacing as `tension = TRUE` across ≥3 operators → cross-universe-pattern
- Same disclosure shape (e.g. "X% deflection rate" disclosed by ≥3 operators) → cross-universe-pattern
- Same regulatory pressure surfacing across multiple regional filers

For each convergence, build a `cross-universe-pattern` lead.

### Step 5: Detect absence signals

Cross-check against:
- `prior_cycle_threads` from prior `earnings-preview` outputs that came due in the lookback window — any unresolved
- Themes that appeared in prior cycles for an operator and are conspicuously missing in the current cycle
- Industry-level commitments (peer announcements, joint initiatives) that should have prompted reactions across the universe but haven't

### Step 6: Detect trajectory shifts

For operators with recent `cycle_snapshots` updates OR substantial post-cycle evidence accumulation:
- Cross-reference any `operator-trajectory-tracker` JSON sidecars from the lookback window
- If the re-classification check recommends SHIFT or REVIEW with strong evidence, build a `trajectory-shift` lead

### Step 7: Pattern-level signals

Distinct from individual leads — these are early-stage convergences that don't yet have a single filing to anchor. Limit to 0–3. For each:
- Describe the pattern
- Name contributing filers with evidence pointers
- Specify the watch trigger that would crystallise it into a commissioning-ready lead

### Step 8: Notable absences

Industry-level absences that deserve flagging even if no single lead covers them. Be honest about which absences are "we missed it" vs. "the news isn't there."

### Step 9: Rank + tier

Rank all leads by editorial value (composite of: novelty, peer-comparability, freshness, ANTA-resonance, format suitability). Assign tiers per the rubric. Cap at the lead-count target.

### Step 10: Methodology + caveats

Lookback boundary, data freshness, any operators/vendors with stale extraction data that suppressed leads.

## Quality checklist

Before delivery:
- [ ] Lookback window stated explicitly
- [ ] Each lead has a specific anchor (named filer or named cross-universe pattern)
- [ ] Each lead has supporting evidence pointers (Supabase row ids or specific filing references), not generic gestures
- [ ] Each lead's `novelty` is honestly assessed; cross-checked against already-commissioned `telecomtv_angles`
- [ ] Tier distribution honest — most leads are `strong-lead` or `worth-considering`, not `must-cover`
- [ ] Suggested next skill is specific (named filer + skill), not vague
- [ ] Pattern-level signals limited to genuine cross-filer convergences (≥3 filers)
- [ ] Notable absences specific, not generic
- [ ] Methodology + caveats note any data-quality issues that affected lead generation
- [ ] No equity-research artefacts (no stock screens, no long/short, no value/growth/quality buckets, no insider trading or short interest)

## Important notes

- This is the editorial pipeline filler — its value compounds over time. Track which leads got commissioned vs. dropped, and refine the scoring rubric accordingly.
- For lead-count target, larger isn't better. 8 sharp leads beats 20 blurry ones. If genuinely fewer signals are present, return fewer leads and say so.
- The "why this didn't surface earlier" field is optional but valuable — if a lead is only visible because the most recent cycle's data just landed, say so. If it's been sitting there and we missed it, say that too.
- For theme-filtered runs (e.g. "AI-RAN only"), the methodology layer doesn't change but the candidate pool is narrower. Still apply the cross-filer pattern detection within the theme.
- Honesty rule: if no lead crosses the `must-cover` threshold this week, don't pad. Say "no must-cover leads this week; strongest is the [lead] in `strong-lead` tier."

## Resources

### ../../references/source-doc-envelope.md
Background on the source_docs envelope shape that the extraction tables follow — relevant context for how this skill walks the evidence graph.

## Dependencies

**Required:**
- `anta-supabase` MCP — for fresh extractions, prior coverage, peer-set context, trajectory state, news_items

**Optional:**
- `telecomtv-archive` MCP — to verify novelty by checking what TelecomTV has already published (more accurate than `telecomtv_angles` review_status alone)
- `peer-set` skill outputs — provide peer-context for "single-filing" leads where peer comparison strengthens the editorial value
- `operator-trajectory-tracker` outputs — feed into `trajectory-shift` lead detection
