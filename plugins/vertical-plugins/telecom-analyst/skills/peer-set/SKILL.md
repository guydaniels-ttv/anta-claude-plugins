---
name: peer-set
description: Construct a peer set for a given operator or vendor — a list of comparable filers with similarity scores, the dimensions of comparison (revenue scale, geographic footprint, business mix, archetype, AI strategy posture), and the rationale for inclusion or exclusion. Distinguishes between scale peers, business-model peers, AI-strategy peers, and regional peers, and lets the user request any combination. Use when the user asks for peer set / comparables / who are [filer]'s peers / who should we benchmark [filer] against / who else looks like [filer] / build a comp set / construct a comparison group.
---

# Peer Set

Construct a defensible peer set for a single operator or vendor — anchored in the ANTA universe, with explicit dimensions of comparison and per-peer similarity scoring. The output is intended both for human editorial use ("who should we benchmark Vodafone against this cycle?") and for downstream Supabase storage so other skills (`telecomtv-angle`, `filing-diff`, `morning-note`) can query peer context cheaply.

This skill is **not** an extraction from a source doc. The output is anchored to the **subject filer** (and optionally an ANTA scoring cycle or a triggering filing), not to a single document. It uses its own `peer_set_run` envelope rather than the `source_doc` envelope used by extraction skills — see [../../references/source-doc-envelope.md](../../references/source-doc-envelope.md) for why.

## When to Use

Use when the user requests:
- "Who are [operator]'s peers?"
- "Build a comp set for [vendor]"
- "Who should we benchmark [filer] against this cycle?"
- "Who else looks like [filer] on AI strategy?"
- "Construct a peer set for [filer] — scale and AI posture"
- "Who else is in the same competitive bracket as [filer]?"

**Do NOT use if:**
- The user wants a one-off comparison of two named filers → just compare them directly, no peer set needed
- The user wants vendor-ecosystem mapping for a specific filer → use `vendor-mentions` instead
- The user wants a sector-wide overview → use `sector-overview-telco` (when authored)

## Inputs

| Input | Required | Notes |
|---|---|---|
| Subject filer | Yes | Name. Cross-check against ANTA universe. |
| Peer dimensions | Optional | Default: `["scale", "geo", "business-mix", "ai-archetype"]`. Other values: `"ai-deployment-stage"`, `"capex-intensity"`, `"shareholder-mix"`. User can pass any subset. |
| Peer set size | Optional | Default 8 peers. Range 3–15. |
| ANTA scoring cycle | Optional | Pass `scoring_cycle_id` to anchor the set to a specific cycle's archetype/trajectory state. Defaults to the most recent completed cycle. |
| Triggering source doc | Optional | If the peer set was triggered by a specific filing (e.g. "Vodafone just released Q3 — who are the peers we should compare them to?"), pass the `source_doc_id`. Goes into `peer_set_run.trigger_source_doc_id`. |

## Output

JSON object with a `peer_set_run` envelope + a `peers` array + a `rationale` array (per-dimension explanation) + a summary narrative + a markdown summary table for human review.

```json
{
  "peer_set_run": {
    "subject_filer_kind": "operator",
    "subject_filer_name": "Vodafone",
    "subject_filer_operator_id": "<uuid from operators table>",
    "scoring_cycle_id": "<uuid or null>",
    "trigger_source_doc_id": "<uuid or null>",
    "dimensions_requested": ["scale", "geo", "business-mix", "ai-archetype"],
    "peer_set_size_target": 8,
    "construction_method": "rules-then-curated",
    "constructed_by": "peer-set v1"
  },
  "peers": [
    {
      "rank": 1,
      "peer_filer_kind": "operator",
      "peer_filer_name": "Orange",
      "peer_filer_operator_id": "<uuid from operators table or null>",
      "similarity_overall": 0.84,
      "similarity_by_dimension": {
        "scale": 0.92,
        "geo": 0.78,
        "business-mix": 0.85,
        "ai-archetype": 0.80
      },
      "shared_dimensions": ["scale", "business-mix", "ai-archetype"],
      "differentiating_factors": ["No Africa exposure (Vodafone has material Vodacom JV)"],
      "rationale_short": "Closest scale peer in EMEA; near-identical fixed+mobile mix; both classified as Strategic Accelerator on the ANTA AI archetype.",
      "in_anta_universe": true,
      "confidence": "high",
      "notes": "Default peer in any Vodafone comp set — present in Q1, Q2, and Q3 2026 baseline runs."
    }
  ],
  "rationale": [
    {
      "dimension": "ai-archetype",
      "subject_value": "Strategic Accelerator (2026-Calibration-May)",
      "scoring_basis": "Match against subject's archetype in the relevant ANTA scoring cycle. Filers with the same archetype score 1.0; adjacent archetypes (e.g. Strategic Accelerator vs AI Vanguard) score 0.6; further-apart archetypes score 0.2.",
      "data_source": "ANTA Supabase cycle_snapshots, cycle 2026-Calibration-May"
    }
  ],
  "candidates_considered_but_excluded": [
    {
      "peer_filer_name": "Comcast",
      "exclusion_reason": "US cable operator; geographic mismatch (no overlap with subject's footprint) and business mix mismatch (no mobile-led core)."
    }
  ],
  "summary": "Eight peers proposed, dominated by EMEA scale operators with similar fixed-mobile mix. Orange and Deutsche Telekom are the canonical comparators — present in every prior Vodafone comp set. Two newer entrants (Telefónica's reshape post-divestments, BT post-Openreach restructure) score above 0.7 on every dimension and should be treated as core peers going forward. One notable exclusion: Comcast — often pulled in for shareholder-return comparisons but geographically and business-mix orthogonal."
}
```

## Peer Dimensions

The dimensions of comparison. The subject's value on each dimension is pulled from `anta-supabase`; the peer's value is pulled the same way; similarity is scored 0..1.

| Dimension | What it measures | Data source |
|---|---|---|
| **scale** | Revenue size, EBITDA size, subscriber base | `operators.annual_revenue`, `operators.subscriber_count` (or comparable fields) |
| **geo** | Geographic footprint overlap | `operators.markets` array, weighted by revenue contribution |
| **business-mix** | Mobile / fixed / B2B / B2C / wholesale revenue mix | `operators.revenue_mix` JSON or comparable |
| **ai-archetype** | ANTA AI archetype assignment for the relevant cycle | `cycle_snapshots.archetype` for the matching `scoring_cycle_id` |
| **ai-deployment-stage** | ANTA AI trajectory + deployment indicator scores | `cycle_snapshots.trajectory` + `scores` for AI deployment indicators |
| **capex-intensity** | Capex / revenue ratio | `operators.capex_intensity` (or computed from `cycle_snapshots`) |
| **shareholder-mix** | State / family / dispersed ownership | `operators.shareholder_mix` |

Score per dimension is 0.0–1.0:
- **1.0** — identical or near-identical
- **0.7–0.9** — same bracket, minor differences
- **0.4–0.6** — adjacent bracket, partial overlap
- **0.1–0.3** — different bracket, weak overlap
- **0.0** — orthogonal

`similarity_overall` is the unweighted mean of `similarity_by_dimension` values for the dimensions the user requested. Don't reweight unless the user asks (and document the weights if you do).

## Construction Methods

| Method | When |
|---|---|
| **rules-only** | Default for ad-hoc requests. Score every operator/vendor in the ANTA universe against the subject on the requested dimensions; take the top N. |
| **rules-then-curated** | Default for editorial use. Run rules-only, then manually adjust the top N to remove obvious mismatches and add canonical peers (peers that are in every prior comp set for this filer, even if a dimension changed). Document the curation in the per-peer `notes`. |
| **embedding** | Reserved for future use when ANTA has filer embeddings. Not yet available — don't use. |

Default to `rules-then-curated`. Only use `rules-only` if the user explicitly requests a strict rules-based result.

## Workflow

### Step 1: Identify the subject

1. Confirm the subject filer; determine `subject_filer_kind` (operator vs. vendor) and look up `subject_filer_operator_id` from `anta-supabase` for operator filers.
2. Confirm the dimensions to use (default `["scale", "geo", "business-mix", "ai-archetype"]` or user-specified subset).
3. Confirm `peer_set_size_target` (default 8).
4. Determine `scoring_cycle_id`: use the user-specified value, or default to the most recent completed cycle in `scoring_cycles`.
5. If a `trigger_source_doc_id` was passed (peer set being built in service of a specific filing), capture it for the envelope.

### Step 2: Pull subject and candidate-pool data

1. Pull the subject filer's values on every requested dimension from `anta-supabase`.
2. Pull the candidate pool — every operator (if subject is operator) or every vendor (if subject is vendor) in the ANTA universe — with the same dimension values.

If a candidate's dimension value is missing in Supabase, exclude it from scoring on that dimension only (don't score 0). If too many candidates are missing too many values, surface that in the summary as a data-quality note.

### Step 3: Score similarity

For each candidate:
1. Score per requested dimension per the rubric above
2. Compute `similarity_overall` as the unweighted mean across requested dimensions
3. Capture `similarity_by_dimension` for transparency

### Step 4: Curate (if `rules-then-curated`)

Pull the most recent prior peer set for the subject from `anta-supabase` (the `peer_set_runs` + `peer_set_memberships` tables). Identify any peers that:
- Are in every recent prior set but didn't make this run's top-N
- Made this run's top-N but are obvious mismatches (unusual archetype combos that don't reflect real comparability)

Manually adjust: re-include canonical peers, drop obvious mismatches. Document each curation step in the affected peer's `notes`.

### Step 5: Build per-peer entries

For each peer in the final set:
- Set `rank`, `similarity_overall`, `similarity_by_dimension`
- List `shared_dimensions` (where the score is ≥0.7) and `differentiating_factors` (one-sentence each, naming the specific deltas)
- Write `rationale_short` — one or two sentences on why this peer made the set
- Set `confidence` — typically `high` for canonical peers, `medium` for newer entrants, `low` for borderline cases pulled in by curation

### Step 6: Build the rationale block

For each requested dimension, populate one entry in `rationale`:
- The subject's value on this dimension (with cycle context where relevant)
- The scoring basis — how scores were assigned
- The data source — which Supabase table/field

This block makes the peer set defensible: anyone reviewing the output can see exactly why each peer scored what they scored.

### Step 7: Capture excluded candidates

For up to 3 candidates that the user might *expect* to see in the set but that didn't make it (e.g. obvious public comparators, peers that were in prior sets but dropped this time), populate `candidates_considered_but_excluded` with a one-sentence exclusion reason.

This is editorially valuable — it pre-empts the "but what about X?" question.

### Step 8: Summary

3–5 sentence editorial summary. Lead with the canonical peers, name the most interesting newer entrant, name the most surprising exclusion. Note any data-quality issues.

## Output Schema

```json
{
  "peer_set_run": {
    "subject_filer_kind": "operator | vendor",
    "subject_filer_name": "string — canonical name from ANTA universe",
    "subject_filer_operator_id": "uuid or null",
    "scoring_cycle_id": "uuid or null",
    "trigger_source_doc_id": "uuid or null",
    "dimensions_requested": ["scale | geo | business-mix | ai-archetype | ai-deployment-stage | capex-intensity | shareholder-mix", ...],
    "peer_set_size_target": "integer 3..15",
    "construction_method": "rules-only | rules-then-curated | embedding",
    "constructed_by": "peer-set v1"
  },
  "peers": [
    {
      "rank": "integer (1 = closest)",
      "peer_filer_kind": "operator | vendor",
      "peer_filer_name": "string — canonical from ANTA universe",
      "peer_filer_operator_id": "uuid or null",
      "similarity_overall": "number 0..1",
      "similarity_by_dimension": {
        "<dimension name>": "number 0..1"
      },
      "shared_dimensions": ["string", ...],
      "differentiating_factors": ["string — one sentence each", ...],
      "rationale_short": "string — one or two sentences",
      "in_anta_universe": "boolean",
      "confidence": "high | medium | low",
      "notes": "string — one sentence; document any curation"
    }
  ],
  "rationale": [
    {
      "dimension": "string — name of the dimension",
      "subject_value": "string — subject's value with cycle context",
      "scoring_basis": "string — how scores were assigned",
      "data_source": "string — Supabase table/field reference"
    }
  ],
  "candidates_considered_but_excluded": [
    {
      "peer_filer_name": "string",
      "exclusion_reason": "string — one sentence"
    }
  ],
  "summary": "string — 3 to 5 sentences"
}
```

## Quality Checklist

Before delivery:
- [ ] Subject filer matched to ANTA canonical name; `subject_filer_operator_id` populated for operator subjects
- [ ] `scoring_cycle_id` set (most-recent-completed default if not user-specified)
- [ ] All requested dimensions actually scored (data missing? exclude that dimension for the affected candidate, don't score 0)
- [ ] `similarity_by_dimension` only contains keys for the dimensions actually requested
- [ ] `similarity_overall` is the unweighted mean across requested dimensions (or weighted with documented weights)
- [ ] Final peer set size matches `peer_set_size_target` (within ±1 if curation forced an adjustment)
- [ ] `rationale` has one entry per requested dimension
- [ ] `candidates_considered_but_excluded` has 0–3 entries; not exhaustive — pick obvious "what about X?" candidates
- [ ] Curated peers (if `rules-then-curated`) are flagged in their `notes`
- [ ] If data quality is degraded (many missing dimension values), said so explicitly in the summary
- [ ] Output JSON is valid and schema-conformant

## Resources

### ../../references/source-doc-envelope.md
Background on why this skill uses its own `peer_set_run` envelope rather than the `source_doc` envelope used by extraction skills.

## Dependencies

**Required:**
- `anta-supabase` MCP — for the operator/vendor universe, dimension values per filer, prior peer sets, and the relevant cycle's archetype/trajectory data

**Optional:**
- `telecomtv-angle` skill — peer-set output feeds the `competitive_context` field of editorial angles
- `filing-diff` skill — peer-set output can scope a "how did this filing differ from peers' equivalent filings?" diff
