---
name: earnings-call-themes
description: Identify and characterise the dominant editorial themes of a single earnings call transcript — what the operator chose to talk about, how analysts pressed back, and what the contrast reveals. Captures both the prepared narrative (CEO/CFO remarks) and the Q&A (where the unscripted signal lives). Each theme gets a category, a salience score, a tension flag (where management framing diverged from analyst pressure), a representative quote from each side, and a one-sentence editorial gist. Use when the user asks to extract themes / dominant topics / editorial angles / what an earnings call was really about / where the tension was / where management and analysts disagreed in a transcript or earnings call.
---

# Earnings Call Themes

Extract the **editorial structure** of a single earnings call transcript — the themes that dominated the conversation, weighted by airtime and emphasis, with explicit attention to where the prepared narrative was challenged or contradicted in Q&A.

This is **not** a fact-extraction skill (`ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker` do that). This is the editorial overview: what the call was about, what management wanted it to be about, and what analysts forced it to be about.

## When to Use

Use when the user requests:
- "What were the main themes of [operator]'s Q3 call?"
- "Pull the editorial themes out of this transcript"
- "Where did management and analysts disagree on this call?"
- "What did [operator] try to talk about and what did analysts force them to talk about?"
- "Give me the editorial angles for this earnings call"

**Do NOT use if:**
- The user wants a single editorial angle for a TelecomTV piece → use `telecomtv-angle`
- The user wants AI-specific content → use `ai-mentions-extractor`
- The source isn't a transcript (filings, press releases) → themes are call-specific; the prepared/Q&A tension structure doesn't transfer
- The user wants a multi-call comparison → use this skill on each call, then `filing-diff --focus themes`

## Inputs

| Input | Required | Notes |
|---|---|---|
| Transcript | Yes | Path or URL to an earnings call transcript. Fetch via `filings-store` MCP if URL. |
| Filer name | If not derivable | Cross-check ANTA universe. |
| Reporting period | If not derivable | E.g. "Q3 2026". Goes into `source_doc.reporting_period`. |
| ANTA scoring cycle | Optional | Pass `scoring_cycle_id` if the run is tied to an ANTA cycle. |

## Output

JSON object with the shared `source_doc` envelope (`doc_type` is always `'earnings_call_transcript'` for this skill) + a `themes` array + a summary narrative + a markdown summary table for human review. The envelope shape is shared with every other extraction skill — see [../../references/source-doc-envelope.md](../../references/source-doc-envelope.md) for the full convention.

```json
{
  "source_doc": {
    "filer_kind": "operator",
    "filer_name": "VEON",
    "filer_operator_id": "<uuid from operators table or null>",
    "doc_type": "earnings_call_transcript",
    "title": "VEON Q3 2026 earnings call transcript",
    "source_url": "https://...",
    "filing_date": "2026-10-30",
    "reporting_period": "Q3 2026",
    "scoring_cycle_id": null,
    "reporting_currency": null,
    "extracted_by": "earnings-call-themes v1"
  },
  "themes": [
    {
      "id": 1,
      "title": "Customer-care GenAI rollout in Pakistan",
      "category": "ai-deployment",
      "salience": "high",
      "share_of_call_pct": 18,
      "prepared_emphasis": "leading",
      "qa_pressure": "moderate",
      "tension": true,
      "tension_summary": "Management led with deflection rate (35%) and revenue-impact framing. Two analysts pressed on whether the deflection was driving CSAT down — management acknowledged 'modest' CSAT pressure but didn't quantify.",
      "management_quote": {
        "speaker": "CEO Kaan Terzioğlu",
        "page_or_timestamp": "37:22",
        "quote": "We've rolled out our self-care GenAI agent in Pakistan, handling 60% of inbound tier-1 queries with a 35% deflection rate from human agents."
      },
      "analyst_quote": {
        "speaker": "Analyst — Goldman Sachs",
        "page_or_timestamp": "1:12:08",
        "quote": "On the Pakistan GenAI agent — what's the CSAT impact looking like, and how are you measuring whether the deflection is real demand reduction vs. customers giving up?"
      },
      "context_tags": ["genai", "customer-care", "pakistan", "csat-tension"],
      "confidence": "high",
      "notes": "Real tension; management framing was clean, analyst pressure was substantive. Worth a TelecomTV angle."
    }
  ],
  "summary": "Five themes dominated. AI deployment (GenAI in Pakistan) was the leading prepared topic and drew real Q&A pressure on CSAT. Capex run-rate was the second-most-pressed theme — management held the line on guidance but conceded mix shift toward GPU spend. Geographic mix (Russia exit progress) got prepared airtime but minimal Q&A — telegraphed as resolved. Two notable absences: no prepared mention of competitive pressure in Bangladesh; no Q&A on tower co valuation despite recent peer transactions."
}
```

## Theme Categories

Open vocabulary, but lean on these conventional buckets first. Add new ones in `notes` if the call genuinely needs them.

| Category | What it covers |
|---|---|
| **financial-performance** | Revenue, margins, EBITDA, cash flow, guidance, FX impact |
| **capex-allocation** | Total capex, capex mix, network capex, AI capex, M&A spend |
| **ai-strategy** | Forward-looking AI plans, partnerships, vision (no in-production references) |
| **ai-deployment** | In-production AI use cases (customer ops, network, field ops) |
| **network-build** | 5G/6G rollout, fibre, fixed-wireless, coverage milestones |
| **competitive-dynamics** | Market share, pricing, churn, new entrants, peer benchmarking |
| **regulatory** | Spectrum, M&A approvals, data protection, AI regulation |
| **m&a-portfolio** | Acquisitions, divestitures, JVs, portfolio rationalisation |
| **shareholder-returns** | Dividends, buybacks, capital structure, leverage |
| **macro-geo** | Country-specific macro, FX, geopolitics, market exit/entry |
| **esg-sustainability** | Energy, carbon, social, governance |
| **other** | Use sparingly — propose a new category in `notes` if you find yourself reaching for `other` |

## Salience and Pressure

**`salience`** — overall importance of the theme on this call.

| Value | Definition |
|---|---|
| **high** | Multiple speakers, multiple Q&A turns, or quantified outcomes referenced |
| **medium** | Substantial single section in prepared remarks or 1–2 Q&A turns |
| **low** | Brief mention, single sentence in prepared remarks, no Q&A pickup |

**`prepared_emphasis`** — how management framed it in prepared remarks.

| Value | Definition |
|---|---|
| **leading** | Opening / headline topic, named in CEO opening or CFO opening |
| **substantive** | Dedicated paragraph or section, but not the lead |
| **passing** | Brief mention, single sentence |
| **absent** | Not mentioned in prepared remarks (only surfaced in Q&A) |

**`qa_pressure`** — how analysts pressed in Q&A.

| Value | Definition |
|---|---|
| **heavy** | 3+ Q&A turns, multiple analysts, follow-up questions |
| **moderate** | 1–2 Q&A turns, one analyst, single follow-up |
| **light** | Single question, no follow-up |
| **absent** | Not raised in Q&A |

**`share_of_call_pct`** — rough estimate (5% increments) of the share of total call airtime devoted to this theme. Sum across all themes should be ≤100% (themes can overlap; allow for unallocated boilerplate / housekeeping).

## Tension

`tension: true` when there is a meaningful gap between the prepared framing and the Q&A line — i.e. analysts pressed for something management didn't volunteer, or contested a framing management offered. This is the editorially valuable signal.

A theme that management framed as "going well" and analysts didn't push on is **not** tension. A theme analysts barely covered is **not** tension. Tension requires both a prepared position AND analyst push that diverges from it.

Capture the tension in `tension_summary` (one or two sentences), and ensure both `management_quote` and `analyst_quote` are populated.

## Workflow

### Step 1: Identify and verify the source

1. Confirm the doc is a transcript (not a written filing). This skill only applies to call transcripts. Filing date captured.
2. Identify the filer; determine `filer_kind` and look up `filer_operator_id` from `anta-supabase` for operator filers.
3. Capture `reporting_period` and `scoring_cycle_id` (if known). `doc_type` is always `'earnings_call_transcript'`. Populate the `source_doc` envelope.

### Step 2: Segment the transcript

Identify the call's structure:
- **Prepared remarks** — typically CEO, then CFO. Note where each ends.
- **Q&A** — list each analyst (firm + name) and which topics each raised.
- **Operator boilerplate / forward-looking statements** — exclude from theme analysis.

### Step 3: Cluster mentions into themes

Read the prepared remarks first. Identify ~5–10 distinct topical threads. For each thread:
- Note the speaker(s)
- Note approximate share of prepared airtime
- Note any quantified claim associated with the theme

Then read Q&A. For each question, note which theme(s) it maps to. Some questions will surface themes that **weren't in prepared remarks** — add these as new themes with `prepared_emphasis: "absent"`.

Aim for 5–10 themes per call. Fewer if the call was tightly scoped; more only if the call had a genuinely sprawling agenda.

### Step 4: Score each theme

For each theme:
1. Set `category` from the conventional bucket list (or open vocabulary if needed)
2. Set `salience`, `prepared_emphasis`, `qa_pressure` per the rubrics above
3. Estimate `share_of_call_pct` (5% increments)
4. Determine `tension` and write `tension_summary` if true
5. Capture the strongest single management quote (`management_quote`) and the strongest single analyst quote (`analyst_quote`) — both with speaker + timestamp + verbatim text (≤500 chars each). Either can be `null` if there genuinely was no quote on that side (but if `tension: true`, both must be populated).
6. Write `context_tags` — lowercase short tags (e.g. `genai`, `pakistan`, `csat-tension`, `5g-rollout`, `dividend-policy`)
7. Write `notes` — one sentence on editorial value or anything non-obvious
8. Set `confidence` (high/medium/low) on the overall theme characterisation

### Step 5: Notable absences

In the `summary`, explicitly call out **what was missing** — themes you would have expected based on the filer's last cycle, peer set, or recent industry events that did not surface on this call. Absences are often the most editorially interesting signal.

### Step 6: Summary

5–8 sentence editorial summary. Lead with the dominant theme, name the most consequential tension, list the notable absences. Don't be exhaustive — pick what an editor would actually want.

## Output Schema

```json
{
  "source_doc": {
    "filer_kind": "operator | vendor",
    "filer_name": "string — canonical name from ANTA universe",
    "filer_operator_id": "uuid or null",
    "doc_type": "earnings_call_transcript",
    "title": "string — human-readable doc title",
    "source_url": "string or null",
    "filing_date": "YYYY-MM-DD or null",
    "reporting_period": "string e.g. 'Q3 2026'",
    "scoring_cycle_id": "uuid or null",
    "reporting_currency": null,
    "extracted_by": "earnings-call-themes v1"
  },
  "themes": [
    {
      "id": "integer",
      "title": "string — short noun phrase, ≤80 chars",
      "category": "financial-performance | capex-allocation | ai-strategy | ai-deployment | network-build | competitive-dynamics | regulatory | m&a-portfolio | shareholder-returns | macro-geo | esg-sustainability | other",
      "salience": "high | medium | low",
      "share_of_call_pct": "integer 0..100, in 5% increments",
      "prepared_emphasis": "leading | substantive | passing | absent",
      "qa_pressure": "heavy | moderate | light | absent",
      "tension": "boolean",
      "tension_summary": "string or null — one or two sentences if tension is true",
      "management_quote": {
        "speaker": "string",
        "page_or_timestamp": "string",
        "quote": "string ≤500 chars, verbatim"
      },
      "analyst_quote": {
        "speaker": "string",
        "page_or_timestamp": "string",
        "quote": "string ≤500 chars, verbatim"
      },
      "context_tags": ["string", ...],
      "confidence": "high | medium | low",
      "notes": "string — one sentence"
    }
  ],
  "summary": "string — 5 to 8 sentences"
}
```

## Quality Checklist

Before delivery:
- [ ] `source_doc` envelope is complete and valid (`doc_type` is `'earnings_call_transcript'`)
- [ ] `filer_operator_id` populated only when `filer_kind == 'operator'` AND a match exists; else null
- [ ] Source is a transcript (not a filing)
- [ ] Both prepared remarks and Q&A have been read in full
- [ ] 5–10 themes identified (not fewer than 4, not more than 12)
- [ ] `share_of_call_pct` values sum to ≤100%
- [ ] Every `tension: true` theme has both `management_quote` and `analyst_quote` populated
- [ ] `tension` is honestly assessed — not every theme has tension
- [ ] At least one notable absence is named in the summary
- [ ] No paraphrased quotes — all are verbatim, ≤500 chars
- [ ] Output JSON is valid and schema-conformant

## Resources

### ../../references/source-doc-envelope.md
The shared `source_doc` envelope convention used across all telecom-analyst extraction skills.

## Dependencies

**Required:**
- `filings-store` MCP — to fetch the transcript
- `anta-supabase` MCP — to look up canonical filer name and prior-cycle themes for cross-comparison

**Optional:**
- `telecomtv-angle` skill — every high-tension or notable-absence theme is a candidate angle; run after this skill to draft editorial pieces
- `ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker` — for the fact-extraction layer beneath themes; run all four on the same transcript for the full editorial + structured-data pass
