---
name: operator-trajectory-tracker
description: Track and update an operator's ANTA archetype + trajectory + composite-score path across cycles. Reviews whether the current archetype/trajectory assignment is still defensible against fresh evidence (extracted filings, themes, vendor moves, AI capex disclosures), surfaces shifts that argue for re-classification at the next scoring cycle, and produces a markdown summary suitable for the ANTA scoring conversation. Use when the user asks to track / review / update / re-check / re-classify an operator's ANTA archetype, trajectory, composite score, or trajectory across cycles, or asks "is the [operator] archetype still right?" / "should we re-classify [operator]?" / "review trajectory for [operator]".
---

# Operator Trajectory Tracker

Maintain and review a single operator's ANTA archetype + trajectory + composite-score path across scoring cycles. The output is a structured review suitable for the ANTA scoring conversation: does the current archetype still hold against accumulated post-cycle evidence, is the trajectory call still right, and what would re-classification require?

This skill is the methodology analogue of "is the thesis still intact" — but the "thesis" here is the ANTA classification (archetype + trajectory) rather than an investment view, and the test is evidence accumulated since the last scoring cycle, not stock-price action.

## When to Use

Use when the user requests:
- "Trajectory check for [operator]"
- "Is [operator]'s archetype still right?"
- "Review [operator]'s ANTA classification"
- "Should we re-classify [operator] at the next cycle?"
- "Update trajectory for [operator] with fresh evidence"
- "Build the steering-board view for [operator]"

**Do NOT use if:**
- The user wants a single-cycle deep dive on one filing → use `earnings-analysis`
- The user wants a peer comparison → use `peer-set`
- The user wants a forward editorial calendar → use `catalyst-calendar`
- The user wants industry-wide context → use `sector-overview-telco`

## Inputs

| Input | Required | Notes |
|---|---|---|
| Operator | Yes | Single operator. Cross-check against the ANTA universe. Vendor subjects are not supported here — vendors don't carry ANTA archetypes. |
| Cycle window | Optional | Default: current cycle + 4 prior cycles (5 total). Specify shorter window if the operator's history is shorter. |
| Audience | Optional | Default `anta-steering-board`. Other values: `analyst-editorial` (for a more story-led framing), `cxo` (for an outward-facing review). |

## Output

Markdown trajectory review with a fixed structure, plus a JSON sidecar capturing the structured review state.

### Markdown structure

```
# [Operator] Trajectory Review — [Date]

**Subject:** [Operator] ([region], tier [1/2])
**Cycle window:** [N cycles, from [oldest] to [latest]]
**Current classification (latest cycle):**
- Archetype: [value]
- Trajectory: [value]
- Composite score: [value]
- DoD: [value]
- Disclosure quality: [value]

## Path summary

[Markdown table — one row per cycle in window]

| Cycle | Archetype | Trajectory | Composite | DoD | Disclosure quality | Notes |
|---|---|---|---|---|---|---|
| 2025-Inception | ... | ... | ... | ... | ... | initial classification |
| 2026-Baseline | ... | ... | ... | ... | ... | |
| 2026-Calibration-May | ... | ... | ... | ... | ... | |

## Movement narrative

[2–4 paragraphs. Walk the path: where the operator started, where it sits now, what the inflection points were. Anchor each major shift to specific evidence (a filing, a partnership announcement, a regulatory event, a capex commitment).]

## Evidence accumulated since the last cycle

[For each major evidence type, list what's accumulated since the last cycle date. Keep tight.

**Filings + transcripts ingested** ([N source_docs since last cycle])
- [date] [doc_type] — one-line note

**AI mentions** ([N rows by category])
- strategy: N | deployment: N | contribution: N | hype: N | risk: N
- Most editorially significant new disclosure: [verbatim quote, source ref]

**AI capex disclosures** ([N new])
- spent: N | committed: N | announced: N | aspirational: N
- Most material new disclosure: [amount + status + source ref]

**Vendor moves**
- New named-strategic relationships: [list]
- Vendors that have gone quiet: [list]

**Earnings-call themes** ([N themes across N transcripts])
- Recurring tension themes: [list]
- New themes vs. prior cycles: [list]
- Themes that have resolved (no longer surfacing): [list]
]

## Re-classification check

For each of the four current classifications (archetype, trajectory, composite trajectory, disclosure quality), assess whether fresh evidence supports the current call:

**Archetype (currently [value])**
- Supports current: [evidence pointing to staying as-is]
- Argues for shift: [evidence pointing to moving — name the candidate target archetype if applicable]
- **Recommendation:** HOLD / REVIEW / SHIFT — short justification

**Trajectory (currently [value])**
- Supports current: ...
- Argues for shift: ...
- **Recommendation:** HOLD / REVIEW / SHIFT

**Composite-score trajectory**
- Indicators showing improvement since last cycle: [list]
- Indicators showing regression: [list]
- Indicators with material new evidence not yet scored: [list]
- **Likely composite movement direction at next cycle:** UP / DOWN / FLAT — with confidence (high/medium/low)

**Disclosure quality (currently [value])**
- Supports current: ...
- Argues for shift: ...
- **Recommendation:** HOLD / REVIEW / SHIFT

## Open questions for the steering board

[2–5 specific questions for the ANTA steering board to resolve at the next cycle. Each anchored to evidence above.]

## Editorial angles

[1–3 short bullets. Even though this skill is methodology-focused, fresh evidence often produces editorial angles. List with `/tv-angle?` flags.]
```

### JSON sidecar

```json
{
  "subject": {
    "filer_kind": "operator",
    "filer_name": "VEON",
    "filer_operator_id": "<uuid>",
    "tier": 1,
    "region": "EMEA + Central Asia"
  },
  "cycle_window": [
    {
      "scoring_cycle_id": "<uuid>",
      "scoring_cycle_name": "2025-Inception",
      "archetype": "Strategic Accelerator",
      "trajectory": "Accelerating",
      "composite_score": 6.2,
      "dod": "Medium",
      "disclosure_quality": "Medium",
      "notes": "string"
    }
  ],
  "evidence_since_last_cycle": {
    "source_docs_count": "integer",
    "ai_mentions_by_category": {"strategy": 0, "deployment": 0, "contribution": 0, "hype": 0, "risk": 0},
    "ai_capex_by_status": {"spent": 0, "committed": 0, "announced": 0, "aspirational": 0},
    "new_named_strategic_vendors": ["string"],
    "vendors_gone_quiet": ["string"],
    "recurring_tension_themes": ["string"],
    "new_themes": ["string"],
    "resolved_themes": ["string"]
  },
  "reclassification": {
    "archetype": {
      "current": "string",
      "recommendation": "HOLD | REVIEW | SHIFT",
      "candidate_target": "string or null",
      "justification": "string"
    },
    "trajectory": {
      "current": "string",
      "recommendation": "HOLD | REVIEW | SHIFT",
      "candidate_target": "string or null",
      "justification": "string"
    },
    "composite_direction": {
      "direction": "UP | DOWN | FLAT",
      "confidence": "high | medium | low",
      "indicators_improving": ["string"],
      "indicators_regressing": ["string"],
      "indicators_with_unscored_evidence": ["string"]
    },
    "disclosure_quality": {
      "current": "string",
      "recommendation": "HOLD | REVIEW | SHIFT",
      "candidate_target": "string or null",
      "justification": "string"
    }
  },
  "open_questions_for_steering_board": ["string"],
  "editorial_angles": ["string"]
}
```

## Workflow

### Step 1: Identify the subject

1. Confirm the operator. If a vendor was passed, refuse and direct the user to a different framing — vendors don't carry ANTA archetypes.
2. Look up `subject_filer_operator_id` from `anta-supabase`.
3. Confirm the cycle window (default: latest 5 cycles).

### Step 2: Pull the path

From `anta-supabase`:
- `cycle_snapshots` rows for the operator across the cycle window — gives archetype, trajectory, composite_score, DoD, disclosure_quality per cycle
- `scoring_cycles` rows joined for cycle dates and names

If the operator has fewer cycles than the window, use what's available and note this.

### Step 3: Pull fresh evidence since the last cycle

Cycle window's most recent cycle has a date. Query for everything ingested since that date for this operator:
- `source_docs` rows (count + types + dates)
- `ai_mentions` rows aggregated by category
- `ai_capex` rows aggregated by disclosure_status (and the most material new amount)
- `vendor_mentions` rows — group by vendor_canonical, identify ones not seen in the prior cycle (new) and ones from prior cycle now absent (gone quiet)
- `earnings_call_themes` rows — group by category, identify recurring tensions and new themes vs. prior cycles
- Any `telecomtv_angles` rows already commissioned for this operator since the last cycle

### Step 4: Walk the movement narrative

Walk through the path table from oldest to newest cycle. For each cycle-to-cycle movement (or stability), explain why anchored to evidence accumulated in the gap.

If this is the operator's first cycle, narrative is just the inception classification justification.

### Step 5: Re-classification check

For each of the four classifications:

1. List evidence supporting holding the current call.
2. List evidence arguing for a shift.
3. Make a recommendation: HOLD (current call is fine), REVIEW (warrants steering-board discussion at next cycle), SHIFT (clear candidate target — name it).

For composite-score direction, this is mechanical: aggregate scored indicators with material new evidence into a likely UP/DOWN/FLAT call. Confidence is honest — `low` when most indicators have ambiguous evidence, `high` when most line up.

### Step 6: Open questions for the steering board

2–5 specific questions, each anchored to evidence above. Examples:
- "Does the Pakistan GenAI deflection (35% Q3, no Q4 update) qualify as a deployment-indicator improvement, given the unresolved CSAT question?"
- "The $750m/4y AI infra commitment was upsized in Q3 — should this re-rate the operator's archetype from Strategic Accelerator to AI Vanguard, or wait for spent-vs-committed evidence?"
- "Vendor reshuffle (Microsoft scope expansion + Algotive newly named) — does this strengthen the Vendor-Ecosystem indicator score or is it too early?"

### Step 7: Surface editorial angles

Even though this skill is methodology-focused, the fresh-evidence walk usually surfaces editorial angles. List 1–3 with `/tv-angle?` flags.

### Step 8: Build the JSON sidecar

Capture the structured review state in the sidecar JSON for downstream consumption (sector-overview-telco's "notable movers" section pulls from this).

## Quality checklist

Before delivery:
- [ ] Subject is an operator (vendors refused with redirect)
- [ ] Path table grounded in actual `cycle_snapshots` rows; missing cycles flagged
- [ ] Evidence-since-last-cycle counts traced to actual table rows (not vibes)
- [ ] Movement narrative anchors every shift to specific evidence
- [ ] Re-classification check has a recommendation (HOLD / REVIEW / SHIFT) for each of the four classifications
- [ ] Composite-direction call has a confidence level
- [ ] Open questions for steering board are specific and anchored to evidence
- [ ] Editorial angles flagged with `/tv-angle?` where applicable
- [ ] No equity-research artefacts (no investment thesis pillars, no stop-loss triggers, no PT or rating)
- [ ] No fabricated indicator scores — only what's in `scores` for the relevant cycles

## Important notes

- This skill is the bridge between editorial signal (from extractions) and methodology (ANTA classifications). The re-classification recommendations are *suggestions* for the steering board, not autonomous decisions. The steering board owns the actual scoring update.
- "Trajectory" here is the ANTA-defined trajectory (Accelerating / Holding / Stalling) — not a financial trajectory and not a stock trajectory.
- For an operator new to the ANTA universe (1 cycle of history), much of the re-classification check will be sparse. Lean into "what evidence has accumulated since inception that should be reflected at the next cycle?"
- For audience = `analyst-editorial`, deprioritise the Open-questions-for-steering-board section and lean into the Editorial-angles section. For `anta-steering-board` audience, do the inverse.

## Resources

### ../../references/source-doc-envelope.md
The shared `source_doc` envelope convention — context for the evidence-since-last-cycle section, which queries source_docs children populated by the extraction skills.

## Dependencies

**Required:**
- `anta-supabase` MCP — for `cycle_snapshots`, `scoring_cycles`, all extraction tables filtered to this operator

**Optional:**
- `peer-set` skill output — useful comparator if the trajectory call would benefit from "how do peers look on the same dimensions?"
- `earnings-analysis` skill — if the user wants the full editorial picture of the latest cycle, run that first and feed the synthesis into this trajectory review
