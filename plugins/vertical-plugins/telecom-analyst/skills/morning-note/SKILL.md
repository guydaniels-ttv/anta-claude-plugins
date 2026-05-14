---
name: morning-note
description: Draft the daily editorial briefing for TelecomTV — a tight, opinionated note covering overnight developments across the ANTA operator and vendor universe. Surfaces newly published filings, fresh disclosures, regulatory news, M&A movement, and editorially significant absences. Designed for a morning editorial conversation, not a trading desk. Use when the user asks for a morning note / daily note / morning briefing / what happened overnight / daily TelecomTV briefing / today's editorial leads.
---

# Morning Note

Draft the daily editorial briefing for TelecomTV. The output is a concise, opinionated note that surfaces what an editor or contributor needs to know before the editorial day starts. Cross-references the ANTA universe, recent extractions, and the catalyst calendar to give a complete picture in under a minute's read.

This skill is **editorial**, not stock-picking. There are no trade ideas, no rating changes, no price targets. The signal is "what should TelecomTV cover today, and what's coming up that needs a hand on it."

## When to Use

Use when the user requests:
- "Morning note for today"
- "What happened overnight?"
- "Daily editorial briefing"
- "Today's morning note for [region]"
- "Run the morning briefing across the ANTA universe"

**Do NOT use if:**
- The user wants a single filing's editorial angles → use `telecomtv-angle`
- The user wants a forward calendar of events (not today) → use `catalyst-calendar`
- The user wants to surface broader story leads (not anchored to overnight news) → use `editorial-leads`

## Inputs

| Input | Required | Notes |
|---|---|---|
| Region scope | Optional | Default: global. Common alternatives: "EMEA only", "APAC only", "US + Canada only". |
| Universe scope | Optional | Default: ANTA tier-1 + tier-2 operators + tracked vendors. |
| Lookback window | Optional | Default: last 24 hours (since the previous morning note). Can extend for Mondays / post-holiday catch-up. |

## Output

Markdown morning-note document with a fixed structure. Designed to be readable in 2 minutes by an editor.

```
# TelecomTV Morning Note — 2026-05-14

## Top story
[2–3 sentences. The single most editorially significant overnight development. Specific, anchored to a named filer and a named disclosure / event.]

## Overnight from the universe

**Operators**
- **[Filer]**: [one-sentence development] — [one-sentence editorial take] (`/tv-angle?` if a follow-up angle is warranted)
- **[Filer]**: ...

**Vendors**
- **[Filer]**: ...

**Regulatory + macro**
- [Region]: [development] — [why it matters for the universe]

## Today's calendar

- [Time] [Event] — [Filer] ([editorial impact: high/medium/low])
- ...

## What we're watching this week

- [Filer Q[X] earnings on [date]] — [one-line setup, drawn from `/earnings-preview` if available]
- [Regulatory deadline] — [filers most exposed]

## Notable absences
- [Operator that should have moved on something they previously committed to but hasn't — name it]
- [Story we'd expect TelecomTV competitors to break by now if it were going to break]

## Editorial actions
- [Filer]: commission `/tv-angle` from yesterday's filing — high-priority Q4 capex disclosure
- [Filer]: `/earnings-preview` due — calls Friday and we have no preview yet
```

After the note, optionally a short **dispatch summary** for distribution: 1-paragraph version of the top story + the 1–3 editorial actions, ready to paste into Slack or email.

## Workflow

### Step 1: Determine scope

1. Confirm region / universe scope (default: global, ANTA tier-1 + tier-2 + vendors).
2. Confirm lookback window (default: 24 hours; extend for Monday / post-holiday).
3. Set the date.

### Step 2: Pull overnight signals

Three sources, in order:

**A. ANTA Supabase — fresh ingests in the lookback window**
Query for rows created since the lookback boundary across:
- `source_docs` — any filings / transcripts / press releases ingested
- `ai_mentions`, `ai_capex`, `vendor_mentions`, `earnings_call_themes`, `telecomtv_angles` — fresh extractions
- `news_items` — anything tagged in the last 24 hours

**B. Filings via WebFetch**
- Any operator IR pages with new releases not yet in `source_docs`
- Any vendor press releases with telco resonance
- Any regulatory bodies (Ofcom, FCC, BNetzA, ARCEP, AGCOM, ANRT, TRAI, etc.) with overnight rulings or consultations

**C. Web search for sector-relevant macro / geo movement**
- Only if it materially affects named operators (e.g. an EU AI Act vote, a major FX move affecting reported numbers, geopolitics affecting a major operator's footprint)
- Don't include generic macro noise

### Step 3: Identify the top story

Pick **one** story for the lead. Criteria:
- Anchored to a specific filer or named event (no generic "5G demand grows" leads)
- Materially new (not a restatement of prior coverage)
- Editorially actionable (could spawn a TelecomTV piece today)

If overnight was genuinely quiet, the top story might be a notable absence ("X was expected to disclose Y this week and hasn't"). That's a valid lead.

### Step 4: Build the universe section

For each filer with an overnight development, write **one sentence** on the development + **one sentence** on the editorial take. Group by operators / vendors / regulatory.

If no development for a filer, don't include them — the section is signal, not census.

If a development warrants a follow-up `/tv-angle`, suffix with `(/tv-angle?)` to flag.

### Step 5: Pull today's calendar

Cross-reference the catalyst calendar for events dated today. List in time order. Mark editorial impact.

### Step 6: Look back through the week

Pull the catalyst calendar for the rest of this week. Identify:
- Earnings calls happening Wed–Fri that need an `earnings-preview` if one isn't already on file
- Regulatory deadlines hitting this week
- Any filer with a multi-day event (investor day, MWC presence) starting this week

### Step 7: Surface notable absences

Cross-check against `prior_cycle_threads` from any recent `/earnings-preview` outputs:
- Any commitment from a prior cycle that should have had a follow-up disclosure by now
- Any operator that committed to a partnership announcement "in the next quarter" and is now past that window

This is the most editorially valuable section over time. Be honest if there are no notable absences worth flagging today.

### Step 8: Write editorial actions

1–3 specific, actionable items for the editorial team:
- Which filings warrant a `/tv-angle` commission today
- Which earnings need a `/earnings-preview` drafted
- Any peer-set work needed for upcoming comparison pieces

### Step 9: Dispatch summary (optional)

If the user asks for a distribution-ready version, condense to one paragraph: top story + the top editorial action. Designed to paste into Slack or an editorial email.

## Quality checklist

Before delivery:
- [ ] Date is today (verified, not from training data)
- [ ] Top story is specific, anchored to a named filer/event, and materially new
- [ ] Every universe-section bullet has both a development AND an editorial take
- [ ] No filer is mentioned without a development — no census filler
- [ ] Today's calendar pulled from `catalyst-calendar` (or built fresh if not loaded)
- [ ] Notable absences honestly assessed — empty section is better than padded
- [ ] Editorial actions are specific (named filer + specific action), not generic ("watch the space")
- [ ] No equity-research artefacts (no trade ideas, no rating changes, no price targets, no PM-speak)
- [ ] Length: ≤1 page when rendered. PMs / editors won't read more.

## Important notes

- Be opinionated — a morning note that just summarises news without an editorial take is wasted work
- "Quiet morning" is a valid morning note — say "limited material overnight; today's focus is [event]"
- Lead with the most important thing — never bury the headline
- Distinguish actionable signal (filings, regulatory rulings, M&A) from noise (vendor blog posts, restated prior coverage)
- Time-stamp your takes — if you're writing at 6am, note that markets and filings may move
- If a prior morning note's editorial actions weren't picked up, briefly nudge in today's "editorial actions" section
- The note should evolve — track patterns across weeks (e.g. "third week running we've flagged X without movement") because that's its own editorial signal

## Dependencies

**Required:**
- `anta-supabase` MCP — for the universe, recent extractions, news_items, and prior morning notes' commitments

**Optional:**
- `WebFetch` — to scan IR pages for overnight releases not yet ingested
- `catalyst-calendar` skill output (loaded earlier in the session) — for today's + this week's events
- `telecomtv-archive` MCP — to flag what TelecomTV has already covered, avoiding duplication
