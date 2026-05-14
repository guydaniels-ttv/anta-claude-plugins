---
name: earnings-preview
description: Build a pre-earnings editorial preview for a single operator or vendor — what to listen for on the call, which prior-cycle disclosures are about to be tested, what AI / capex / vendor-ecosystem questions are likely to come up, and which prior-cycle tensions are heading for resolution. Designed for a TelecomTV editor pre-positioning coverage and an ANTA contributor anticipating evidence flow into the active scoring cycle. Use when the user asks for an earnings preview / pre-earnings note / what to listen for / what to watch on [filer]'s upcoming call / prep for [filer] Q[X].
---

# Earnings Preview

Generate a pre-earnings editorial preview for a single operator or vendor — anchored in their most recent prior cycle's disclosures and the open editorial questions those disclosures created. Output is a focused "what to listen for" note, not a financial-model scenario builder.

This skill is **editorial / methodology-driven**, not stock-picking. There are no consensus estimates, no bull/bear scenarios with stock-price implications, no whisper numbers. The signal is "what would TelecomTV actually want to cover from this call, and which prior-cycle threads need resolving."

## When to Use

Use when the user requests:
- "Earnings preview for [operator]'s next results"
- "What should we listen for on [vendor]'s Q3 call?"
- "Pre-earnings note on [filer]"
- "Prep me for [filer]'s upcoming earnings"
- "What tensions from [filer]'s last call need resolution?"

**Do NOT use if:**
- The call has already happened → use `earnings-call-themes` + `telecomtv-angle` for post-call work
- The user wants a forward calendar across many filers → use `catalyst-calendar`
- The user wants the daily editorial briefing → use `morning-note`

## Inputs

| Input | Required | Notes |
|---|---|---|
| Filer | Yes | Operator or vendor name. Cross-check against the ANTA universe. |
| Reporting period | If not derivable | E.g. "Q3 2026", "FY 2025". The period the filer is *about to* report. |
| Audience hint | Optional | Default `analyst-editorial`. Same vocabulary as `telecomtv-angle`. |

## Output

JSON object with the shared `source_doc` envelope (`source_doc.doc_type` is always `'earnings_call_transcript'` because the preview *anticipates* a transcript) + a `listen_for` array + a `prior_cycle_threads` array + a markdown summary table + a 4–6 sentence editorial summary.

```json
{
  "source_doc": {
    "filer_kind": "operator",
    "filer_name": "VEON",
    "filer_operator_id": "<uuid from operators table or null>",
    "doc_type": "earnings_call_transcript",
    "title": "VEON Q4 2026 earnings call (anticipated)",
    "source_url": null,
    "filing_date": "2027-02-15",
    "reporting_period": "Q4 2026",
    "scoring_cycle_id": null,
    "reporting_currency": "USD",
    "extracted_by": "earnings-preview v1"
  },
  "listen_for": [
    {
      "id": 1,
      "topic": "Pakistan customer-care GenAI deflection trajectory",
      "category": "ai-deployment",
      "priority": "high",
      "why": "Q3 disclosure was 35% deflection rate; analysts pressed on CSAT impact and management hedged. A Q4 update on either deflection trend or CSAT measurement would close the loop on the most-pressed Q3 thread.",
      "questions_to_listen_for": [
        "Has the deflection rate held / improved / regressed in Q4?",
        "Has management quantified CSAT impact as they were pressed to in Q3?",
        "Has the Pakistan deployment scope been extended to other markets?"
      ],
      "speaker_to_watch": "CEO Kaan Terzioğlu (in prepared remarks); CFO if pressed on cost savings"
    }
  ],
  "prior_cycle_threads": [
    {
      "id": 1,
      "thread": "Three-year AI infra commitment scaled up",
      "prior_cycle": "Q3 2026",
      "prior_quote": "We are now committing $750m over four years to our AI infrastructure programme.",
      "what_to_watch": "Q4 disclosure of any spent-vs-committed update; whether the run-rate ($187m/yr implied) is reflected in the FY27 capex guidance",
      "resolves_if": "Concrete Q4 spent figure given OR FY27 capex guide explicitly references the AI infra programme"
    }
  ],
  "summary": "VEON Q4 preview is dominated by two threads from Q3: the Pakistan GenAI deflection-vs-CSAT tension (highest editorial priority — needs either a fresh deflection number or a CSAT data point) and the upsized $750m/4y AI infra commitment (needs a spent / committed update or FY27 capex guide reference). Secondary watch items: any vendor reshuffle following the Microsoft Azure scope expansion announced in Q3, and any update on the Bangladesh RAN energy-optimisation pilot (mentioned passing in Q3 with new vendor name Algotive). No quantified guidance has been given for Q4 itself; the call will likely be the first time investors see the FY27 outlook."
}
```

## Categories

`listen_for` items use the same 12-value `category` enum as `earnings-call-themes`:
`financial-performance | capex-allocation | ai-strategy | ai-deployment | network-build | competitive-dynamics | regulatory | m&a-portfolio | shareholder-returns | macro-geo | esg-sustainability | other`

`priority`: `high` (likely a TelecomTV piece on its own) / `medium` (worth a roundup mention) / `low` (background context only).

## Workflow

### Step 1: Identify the filer + period

1. Confirm the filer; determine `filer_kind` and look up `filer_operator_id` from `anta-supabase`.
2. Confirm the upcoming `reporting_period`. If the user said "Q4" without a year, infer from the most recent prior period in `source_docs`.
3. Find the call's expected date (IR page, web search, or `catalyst-calendar` output if available). Set `source_doc.filing_date` to the anticipated date (or null if unknown).
4. Populate the `source_doc` envelope. `doc_type` is `'earnings_call_transcript'`.

### Step 2: Pull prior-cycle context from Supabase

Query `anta-supabase` for this filer's most recent prior cycle:
- `earnings_call_themes` rows — especially any `tension = TRUE` rows (these are the threads needing resolution)
- `ai_mentions` rows in the `strategy` / `deployment` / `contribution` categories (forward-looking strategy claims that should now be tested against execution)
- `ai_capex` rows with `disclosure_status` of `committed` or `announced` (commitments that should now have a spent / progress update)
- `vendor_mentions` rows with `materiality = 'named-strategic'` (relationships likely to surface again)
- Any `telecomtv_angles` rows with `review_status = 'commissioned'` or `'published'` from the prior cycle (so the preview reflects what TelecomTV has already covered)

If `anta-supabase` has no prior-cycle data for this filer, say so — the preview will be thinner.

### Step 3: Build the `listen_for` items

Aim for 3–7 `listen_for` items. Each one:
- Anchored to a category from the enum
- Has a clear `why` — usually traceable to a specific prior-cycle row (theme tension, commitment, vendor relationship)
- Has 2–4 specific questions an editor could match against the actual call once it lands
- Names a likely speaker (CEO for strategy, CFO for capex / financials, segment head for operational metrics)

Don't pad the list. Three sharp items beats seven bland ones.

### Step 4: Build the `prior_cycle_threads`

For every `committed` or `announced` AI capex disclosure, every `tension = TRUE` theme, and every named-strategic vendor relationship from the prior cycle, create a `prior_cycle_threads` entry that captures:
- The thread (one sentence)
- The prior cycle's verbatim quote (≤500 chars, from the Supabase row)
- What to watch for that would resolve / extend / undermine the thread
- A `resolves_if` line — a falsifiable condition that, if met by the upcoming call, closes the thread

This is the editorial backbone: the post-call work (`earnings-call-themes` + `telecomtv-angle`) can then check off threads against the resolves_if conditions.

### Step 5: Write the summary

4–6 sentence editorial summary. Lead with the single most editorially loaded thread. Name the highest-priority `listen_for` item. Note any secondary watch items. Be honest if there isn't much in prior cycles to anchor against (some filers have thin disclosure history).

### Step 6: Cross-check the catalyst calendar

If the catalyst calendar has a `notes` line for this filer's results date, ensure consistency. Flag any mismatch (e.g. catalyst calendar says May 14 but IR page says May 20).

## Quality checklist

Before delivery:
- [ ] `source_doc` envelope populated; `doc_type` is `'earnings_call_transcript'`
- [ ] `filer_operator_id` populated only when `filer_kind == 'operator'` AND a match exists; else null
- [ ] Anticipated `filing_date` captured (or explicitly null)
- [ ] 3–7 `listen_for` items; each has a category, priority, why, and 2–4 specific questions
- [ ] Every `prior_cycle_thread` has a verbatim prior quote (≤500 chars), from `anta-supabase`, with a falsifiable `resolves_if` condition
- [ ] Priority distribution honest — not everything is `high`
- [ ] If prior-cycle data is thin in Supabase, said so explicitly
- [ ] No equity-research artefacts (no consensus estimates, no whisper numbers, no bull/bear scenarios with stock-price reaction)

## Important notes

- This skill produces a *preview* — the output is anchored to a transcript that hasn't happened yet. Once the transcript lands, the natural follow-on is `earnings-call-themes` + `telecomtv-angle`, and the `resolves_if` conditions become the checklist.
- The `source_doc` envelope on this preview is *anticipatory*. The loader can still UPSERT into `source_docs` (`filing_date` will be the anticipated date, replaced with the actual date once the call lands) but should NOT ingest `listen_for` or `prior_cycle_threads` into the post-call extraction tables — these are pre-call artefacts.
- If a filer has thin Supabase history (new universe entry, recent first cycle), the preview will skew toward forward questions rather than thread resolution. Flag this.

## Resources

### ../../references/source-doc-envelope.md
The shared `source_doc` envelope convention used across all telecom-analyst extraction skills.

## Dependencies

**Required:**
- `anta-supabase` MCP — for prior-cycle themes, capex disclosures, vendor mentions, and angle history

**Optional:**
- `filings-store` MCP — to fetch the filer's IR-calendar page for the anticipated date
- `catalyst-calendar` skill — to ensure the anticipated date matches the calendar
- `telecomtv-archive` MCP — to pull prior published TelecomTV coverage of this filer
