---
description: Build a forward editorial calendar of upcoming catalysts across the ANTA operator + vendor universe (earnings, regulatory deadlines, MWC and other industry events, investor days, M&A milestones)
argument-hint: "[time horizon, e.g. 'next 2 weeks' or 'until MWC'] [universe scope, optional]"
---

# Catalyst Calendar Command

Build the editorial catalyst calendar for the ANTA operator + vendor universe over a forward horizon. Prioritised for TelecomTV editorial commissioning + ANTA scoring-cycle awareness.

## Workflow

### Step 1: Parse the inputs

- **Time horizon** — optional; default next 2 weeks. Common alternatives: "next month", "next quarter", "until MWC 2027".
- **Universe scope** — optional; default full ANTA universe. Can scope to a region, tier, or named subset (e.g. "tier-1 EMEA only").
- **Event-type filter** — optional; default all. Can scope to e.g. "earnings only" or "regulatory + spectrum only".

### Step 2: Verify and load

1. Pull the operator + vendor universe in scope from `anta-supabase`.
2. Pull recent `source_docs` rows for each filer to estimate next reporting periods.
3. If `telecomtv-archive` MCP is available, prepare to cross-check against already-commissioned coverage.

### Step 3: Invoke the skill

Use `skill: "catalyst-calendar"` to gather events, score editorial impact, and surface absences.

### Step 4: Deliver output

Provide three things:

1. **Markdown calendar table** sorted by date asc — date / event / filer / type / editorial impact / notes
2. **Tier list** — "this week" / "next week" / "later" with one-sentence editorial framing per high-impact event
3. **Notable absences** — events you'd expect to see but can't find dates for; these are editorial signals in their own right

### Step 5: Offer next steps

After delivering, offer:
- "Want me to draft `/morning-note` framing for this week's high-impact events?"
- "Want me to start `/earnings-preview` for the upcoming operator results?"
- "Want me to flag any events as evidence triggers for the next ANTA scoring cycle?"

## Quality Checklist

Before delivery, confirm:
- [ ] Every event date verified, not inferred from training data
- [ ] Every filer mapped to a canonical ANTA-universe name; `filer_kind` set
- [ ] `editorial impact` distribution is honest — not everything is `high`
- [ ] Industry-event dates correct for the year (MWC etc. shift annually)
- [ ] Notable absences explicitly listed in the summary
- [ ] No equity-research artefacts (no consensus estimates, price targets, buy/sell calls)
