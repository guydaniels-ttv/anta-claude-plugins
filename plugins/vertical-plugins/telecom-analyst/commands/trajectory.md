---
description: Track and review an operator's ANTA archetype + trajectory + composite-score path; assess whether the current classification is still defensible against fresh post-cycle evidence; produce a steering-board-ready review
argument-hint: "[operator name] [cycle window, optional, default 5 cycles]"
---

# Operator Trajectory Tracker Command

Build the methodology-layer review for a single operator's ANTA classification across cycles. Anchored in `cycle_snapshots` + post-cycle accumulated evidence from the extraction tables.

## Workflow

### Step 1: Parse the inputs

- **Operator name** — required. Cross-check against the ANTA universe. Vendors are not supported (vendors don't carry ANTA archetypes).
- **Cycle window** — optional; default latest 5 cycles. Specify shorter window if history is shorter.
- **Audience** — optional; default `anta-steering-board`. Other values: `analyst-editorial`, `cxo`.

If the operator is missing, ask:
- "Which operator should I review the trajectory for?"

If a vendor is given, redirect:
- "Vendors don't carry ANTA archetypes. Did you mean to ask about an operator's vendor relationships? Use `/vendors` instead."

### Step 2: Verify and load

1. Look up `subject_filer_operator_id` in `anta-supabase`.
2. Pull `cycle_snapshots` for the operator across the cycle window.
3. Pull all extraction-table rows for the operator since the last cycle date (`source_docs`, `ai_mentions`, `ai_capex`, `vendor_mentions`, `earnings_call_themes`).
4. Pull `scores` rows for the operator across the window for indicator-level analysis.
5. Pull any `telecomtv_angles` already commissioned/published for the operator since the last cycle.

### Step 3: Invoke the skill

Use `skill: "operator-trajectory-tracker"` to assemble the review.

### Step 4: Deliver output

Provide:

1. **Markdown trajectory review** with the fixed structure: subject snapshot / path summary / movement narrative / evidence accumulated since last cycle / re-classification check / open questions for steering board / editorial angles.
2. **JSON sidecar** capturing the structured review state — for downstream consumption by `sector-overview-telco`'s "notable movers" section.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to run `/peer-set` for this operator to compare trajectory against peers?"
- "Want me to commission `/tv-angle` drafts for the editorial angles surfaced?"
- "Want me to package the open steering-board questions into a memo format?"
- "Want me to run `/filing-diff` against the operator's last two cycles to deepen the movement narrative?"

## Quality Checklist

Before delivery, confirm:
- [ ] Subject is an operator (vendor refused with redirect)
- [ ] Subject matched to ANTA canonical name; `subject_filer_operator_id` populated
- [ ] Path table grounded in actual `cycle_snapshots` rows; missing cycles flagged
- [ ] Evidence-since-last-cycle counts traced to actual table rows
- [ ] Movement narrative anchors every shift to specific evidence
- [ ] Re-classification check has HOLD/REVIEW/SHIFT recommendation for each of the four classifications
- [ ] Composite-direction call has a confidence level
- [ ] Open steering-board questions specific and evidence-anchored
- [ ] Editorial angles flagged where applicable
- [ ] No equity-research artefacts (no investment thesis pillars, no stop-loss triggers, no PT or rating)
- [ ] No fabricated indicator scores
