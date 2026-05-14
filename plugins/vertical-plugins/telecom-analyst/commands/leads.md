---
description: Generate ranked editorial story leads across the ANTA universe — single-filing leads, cross-universe patterns, absence signals, trajectory shifts, vendor ecosystem moves
argument-hint: "[lookback window, optional, default '14 days'] [theme filter, optional, e.g. 'AI-RAN only'] [universe scope, optional]"
---

# Editorial Leads Command

Surface ranked editorial story leads across the ANTA operator + vendor universe. The editorial pipeline filler — what should TelecomTV be writing about that we haven't already commissioned?

## Workflow

### Step 1: Parse the inputs

- **Lookback window** — optional; default 14 days. Common alternatives: "since last cycle", "last 30 days", "since [named event]".
- **Theme filter** — optional; restricts leads to those touching the theme (e.g. "AI-RAN only").
- **Universe scope** — optional; default full ANTA universe.
- **Lead count target** — optional; default 10. Range 3–25.
- **Audience** — optional; default `analyst-editorial`.

### Step 2: Verify and load

1. Pull all in-window source_docs + child extraction rows from `anta-supabase` for the universe in scope.
2. Pull `telecomtv_angles` rows in the window with `review_status` of `commissioned` or `published` to avoid recommending duplicates.
3. Pull recent `peer_set_runs` and any `operator-trajectory-tracker` JSON sidecars for context.
4. Pull recent `earnings-preview` outputs to identify prior-cycle threads that came due.
5. If `telecomtv-archive` MCP is available, prepare for novelty cross-check.

### Step 3: Invoke the skill

Use `skill: "editorial-leads"` to score, rank, and tier the leads.

### Step 4: Deliver output

Provide:

1. **Markdown ranked leads list** with the fixed structure: top leads (each with anchor, type, story, evidence, novelty, format, suggested next skill) / pattern-level signals / notable absences / methodology + caveats.
2. **JSON sidecar** with the full structured leads list — for tracking commissioning rate over time.

### Step 5: Offer next steps

After delivering, offer (named specifically per the highest-tier leads):
- "Want me to draft the full `/tv-angle` for [lead 1]?"
- "Want me to run `/peer-set` for [filer in lead 2] to deepen the peer-context?"
- "Want me to track which of these get commissioned vs. dropped for next week's run?"
- "Want me to filter the leads to just `must-cover` tier for the editorial Slack?"

## Quality Checklist

Before delivery, confirm:
- [ ] Lookback window stated explicitly
- [ ] Each lead has a specific anchor (named filer or named pattern)
- [ ] Each lead has supporting evidence pointers (Supabase row ids or specific filing references)
- [ ] Each lead's `novelty` honestly assessed; cross-checked against already-commissioned `telecomtv_angles`
- [ ] Tier distribution honest — most leads are `strong-lead` or `worth-considering`, not `must-cover`
- [ ] Suggested next skill specific (named filer + skill), not vague
- [ ] Pattern-level signals limited to genuine ≥3-filer convergences
- [ ] Notable absences specific, not generic
- [ ] Methodology + caveats note any data-quality issues
- [ ] No equity-research artefacts (no stock screens, no long/short, no value/growth/quality buckets, no short-interest framing)
