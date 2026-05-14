---
name: catalyst-calendar
description: Build and maintain a forward calendar of editorial catalysts across the ANTA operator and vendor universe — earnings dates, investor days and capital-markets days, MWC and other industry shows, regulatory deadlines (spectrum auctions, M&A approvals, AI Act milestones), product / network launches, and macro events with telco resonance. Helps a TelecomTV editor or ANTA contributor prioritise attention and pre-position coverage. Use when the user asks for a catalyst calendar / event calendar / earnings calendar / what's coming up / what to watch this week / next month / quarter / upcoming events for [operator or vendor].
---

# Catalyst Calendar

Maintain a forward editorial calendar of upcoming catalysts across the ANTA universe. The output is a prioritised, dated event list that a TelecomTV editor can use for commissioning and an ANTA contributor can use to anticipate evidence flow into the scoring cycle.

This skill is **editorial / methodology-driven**, not stock-picking. Catalysts are anchored to operators or vendors in the ANTA universe and are scored on editorial impact, not stock-price reaction.

## When to Use

Use when the user requests:
- "Catalyst calendar for the next two weeks"
- "What's coming up across the operator universe?"
- "Earnings calendar for tier-1 operators"
- "Upcoming events for [filer]"
- "What should we be watching this quarter?"
- "Build the catalyst calendar for the run-up to MWC"

**Do NOT use if:**
- The user wants editorial story angles for *one specific* event → use `telecomtv-angle` after the event lands
- The user wants a single operator's trajectory rather than forward events → use `operator-trajectory-tracker`
- The user wants the daily editorial briefing → use `morning-note`

## Inputs

| Input | Required | Notes |
|---|---|---|
| Time horizon | Optional | Default: next 2 weeks. Common alternatives: next month, next quarter, run-up to a named event (e.g. "until MWC 2027"). |
| Universe scope | Optional | Default: all of the ANTA tier-1 + tier-2 operator universe + tracked vendors. Can scope to a region, tier, or named subset. |
| Event types | Optional | Default: all categories below. Can scope to e.g. "earnings only" or "regulatory + spectrum only". |

## Output

Markdown calendar table sorted by date asc, plus a narrative summary, plus a prioritised "this week" / "next week" / "later" tier list.

```
| Date       | Event                                  | Filer / Subject     | Type        | Editorial impact | Notes |
|------------|----------------------------------------|---------------------|-------------|------------------|-------|
| 2026-05-20 | Vodafone FY26 results + earnings call  | Vodafone (operator) | earnings    | high             | First disclosure on AI capex post-Microsoft restructure |
| 2026-05-22 | Ofcom spectrum 6 GHz consultation close | UK regulator        | regulatory  | medium           | Affects all UK MNOs; vendor implications for NEPs |
| 2026-06-04 | Ericsson investor briefing on AI-RAN    | Ericsson (vendor)   | corporate   | high             | Likely AI-RAN partnership announcements |
| 2026-06-15 | T-Mobile US Q2 results                  | T-Mobile (operator) | earnings    | medium           | Watch GenAI customer-care metrics post-Q1 disclosure |
| ...                                                                                                              |
```

Each event row carries:
- `date` (ISO YYYY-MM-DD; for ranges, the start date with end in `notes`)
- `event` — short descriptive title
- `filer` — canonical name from the ANTA universe + (operator|vendor) discriminator
- `type` — one of: `earnings`, `corporate` (investor day, CMD, M&A milestone, management transition), `regulatory` (spectrum, AI Act, M&A approval, data-protection ruling), `industry` (MWC, ITU, conferences), `macro` (rate decisions, FX events with named telco resonance only)
- `editorial impact` — `high` / `medium` / `low` — based on editorial newsworthiness for TelecomTV's audience, not stock impact
- `notes` — one short editorial line. Why does an editor care?

After the table:

**This week's high-impact events** — 1–3 dated bullets with a one-sentence editorial framing per event.

**Next week's heads-up** — 1–3 bullets on what's coming.

**Later but worth flagging** — 0–3 bullets on events further out that are big enough to start preparing for now.

**Notable absences** — events you'd expect to see but can't find dates for (e.g. operator that should have announced its results date by now and hasn't). These are themselves editorial signals.

## Event categories

| Category | What it covers |
|---|---|
| **earnings** | Quarterly / half-year / full-year results + earnings call dates |
| **corporate** | Investor day, capital markets day, analyst day, management transitions, M&A milestones (signing, regulatory approval, close), divestiture completions, debt refinancing of editorial size |
| **regulatory** | Spectrum auctions / consultation deadlines, M&A approval decisions, AI Act milestones, data-protection rulings, telecoms code amendments, net-neutrality reviews |
| **industry** | MWC (Barcelona, Las Vegas, Shanghai), TM Forum, DTW, NetworkX, Big 5G Event, ITU events, Open RAN Alliance plenaries, hyperscaler conferences with telco tracks (Microsoft Ignite, AWS re:Invent, Google Cloud Next) |
| **corporate launches** | Major network launches (5G-Advanced, AI-RAN deployments at scale), product launches with sector resonance, partnership announcements with capex disclosure |
| **macro** | Only telco-resonant — central-bank moves affecting telco capex cycles, FX events affecting reported numbers materially, geopolitics affecting major operators (Russia exits, MEA M&A) |

## Editorial impact rubric

`high` — drives a likely TelecomTV piece on its own; or sets up a tension for an `/tv-angle` follow-up; or provides material evidence into the next ANTA scoring cycle.

`medium` — worth covering but not a standalone story; usually combined with adjacent events into a roundup.

`low` — calendar completeness only; no commissioning trigger expected.

Be honest. Most quarterly results are `medium` (everyone reports them). What makes one `high` is novel disclosure expected, a peer comparison opportunity, or a pre-existing tension waiting to be resolved.

## Workflow

### Step 1: Determine scope

1. Confirm time horizon (default: next 2 weeks).
2. Confirm universe scope (default: full ANTA universe; scope down on request).
3. Confirm event-type filter (default: all categories).

### Step 2: Pull the universe

Query `anta-supabase` for the operator + vendor universe in scope. Capture canonical names + tier (T1/T2 for operators) + region.

### Step 3: Gather earnings dates

For each operator and vendor in scope:
- Query the most recent ANTA `source_docs` row for the previous reporting period — the next reporting period date is typically ~13 weeks later
- For operators with published IR calendars, prefer those (note the source URL in `notes`)
- Web-search current published dates for the top names; flag any that haven't yet announced

### Step 4: Gather corporate + regulatory + industry events

- **Industry events** are stable across the year — pull from a maintained list or web search by name + year (MWC 2027 dates etc.)
- **Regulatory deadlines** require region-specific search per relevant regulator (Ofcom, BNetzA, FCC, ARCEP, AGCOM, ANRT, TRAI)
- **Corporate events** (investor days, M&A milestones) require per-filer web search; use the ANTA news_items table if recent flagging exists

### Step 5: Score editorial impact

For each event, set `editorial impact` per the rubric. Don't rate everything `high` — the value of the calendar is the prioritisation.

### Step 6: Look for absences

Cross-check the universe against the populated calendar:
- Tier-1 operators not represented in the horizon → did you miss their date, or have they not announced one?
- Reporting periods that should be falling in the horizon but aren't → flag

Both kinds of absence become editorial leads in their own right.

### Step 7: Write the tier list + summary

Group the events into "this week" / "next week" / "later" tiers and write 1–3 bullets per tier with editorial framing. Keep short — an editor should scan this in under a minute.

## Quality checklist

Before delivery:
- [ ] Every event has a verified date (not inferred from training data)
- [ ] Every filer mapped to a canonical name in the ANTA universe; `filer_kind` set
- [ ] `editorial impact` distribution is honest — most events are not `high`
- [ ] Notable absences explicitly listed in the summary
- [ ] Industry events are dated correctly for the year (MWC etc. shift)
- [ ] Regulatory deadlines verified against the regulator's published calendar (not a secondary source)
- [ ] No US-equity-research artefacts (no consensus estimates, no price targets, no buy/sell calls)

## Important notes

- Earnings dates shift — verify against the operator's IR page closer to the date
- Some catalysts are recurring (monthly traffic data from regulators) — note these as a class once rather than per-instance
- Conference attendance / speaker lists matter — which operators are presenting at MWC, and which are conspicuously absent? This is its own editorial angle.
- Archive past catalysts with the actual outcome — feeds into `editorial-leads` pattern recognition over time
- For ANTA-cycle-relevant events, set the `notes` field to flag which scoring cycle they feed (e.g. "evidence for 2026-Calibration-May")

## Dependencies

**Required:**
- `anta-supabase` MCP — for the canonical operator + vendor universe and prior-cycle source_docs

**Optional:**
- `WebFetch` — to fetch IR-calendar pages for operators
- `telecomtv-archive` MCP — to flag editorial coverage already commissioned for events on the calendar
