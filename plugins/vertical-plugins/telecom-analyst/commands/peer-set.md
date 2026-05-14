---
description: Construct a defensible peer set for an operator or vendor — top N comparables with per-dimension similarity scoring, rationale, and notable exclusions
argument-hint: "[subject filer name] [dimensions, comma-separated, optional] [peer set size, optional]"
---

# Peer Set Command

Build a peer set anchored in the ANTA universe. Produces a top-N peer list with similarity scores, dimension-by-dimension rationale, and a "candidates considered but excluded" block to pre-empt the obvious "what about X?" question.

## Workflow

### Step 1: Parse the inputs

- **Subject filer** — name of the operator or vendor the peer set is for. Required.
- **Dimensions** — optional, comma-separated subset of `scale, geo, business-mix, ai-archetype, ai-deployment-stage, capex-intensity, shareholder-mix`. Default: `scale, geo, business-mix, ai-archetype`.
- **Peer set size** — optional integer 3–15. Default: 8.
- **ANTA scoring cycle** — optional; pass `scoring_cycle_id` to anchor archetype/trajectory comparisons to a specific cycle. Defaults to the most recent completed cycle.
- **Triggering source doc** — optional; pass `source_doc_id` if the peer set is being built in service of a specific filing.

If the subject filer is missing, ask:
- "Which filer should I build a peer set for?"

### Step 2: Verify and load

1. Look up the subject filer in `anta-supabase` — canonical name + `subject_filer_operator_id` (operator subjects only).
2. Confirm the cycle: if not user-specified, query the most recent completed `scoring_cycles` row.
3. Pull the candidate pool from `anta-supabase` — every operator (if subject is operator) or every vendor (if subject is vendor).
4. Pull the most recent prior peer set for the subject (if any) — used for the curation step.

### Step 3: Invoke the skill

Use `skill: "peer-set"` to do the scoring, curation, and rationale construction.

### Step 4: Deliver output

Provide three things:

1. **Markdown summary table** sorted by `rank` asc:

   | Rank | Peer | Overall | Per-dimension scores | Shared dims | Differentiating | Confidence |
   |---|---|---|---|---|---|---|

2. **JSON object** matching the schema in the skill's `SKILL.md` — `peer_set_run` envelope + `peers` array + `rationale` array + `candidates_considered_but_excluded` + `summary` — ready for Supabase ingest.

3. **3–5 sentence narrative** for an editor or analyst. Lead with the canonical peers, name the most interesting newer entrant, name the most surprising exclusion. Note any data-quality issues.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to ingest this into the ANTA Supabase `peer_set_runs` + `peer_set_memberships` tables?" (Pending table design — flag if not yet available.)
- "Want me to compare [subject]'s most recent filing to each peer's equivalent filing via `/filing-diff`?"
- "Want me to use this peer set to widen the `competitive_context` of an existing `/tv-angle` output?"
- "Want me to score the same peer set on a different cycle to see how it has shifted?"

## Quality Checklist

Before delivery, confirm:
- [ ] Subject filer matched to ANTA canonical name; `subject_filer_operator_id` populated for operator subjects
- [ ] `scoring_cycle_id` is set (most-recent-completed if not user-specified)
- [ ] Only the requested dimensions appear in `similarity_by_dimension`
- [ ] `similarity_overall` is the documented mean of requested dimensions
- [ ] Final peer count matches the requested size (±1 if curation forced an adjustment)
- [ ] `rationale` has one entry per requested dimension with `data_source` referenced
- [ ] `candidates_considered_but_excluded` populated with 0–3 obvious-but-excluded names
- [ ] Curated additions/removals (if `rules-then-curated`) are flagged in the per-peer `notes`
- [ ] Data-quality issues (many missing dimensions) called out in the summary
- [ ] Output JSON validates against the schema
