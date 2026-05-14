---
name: earnings-analysis
description: Produce a post-call editorial analysis of an operator or vendor's just-reported quarterly results — synthesises the structured extractions from `ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker`, and `earnings-call-themes` into a single editorial deliverable that resolves prior-cycle threads, lands the editorial top story, and surfaces commissioning-ready angles. Designed for TelecomTV editorial output, not equity-research valuation. Use when the user asks to analyse / write up / cover / report on / produce post-earnings analysis for an operator or vendor's just-reported quarter / FY / H1 results.
---

# Earnings Analysis

Produce a post-call editorial write-up for a single operator or vendor's just-reported quarterly results. The output is the editorial layer that sits on top of the structured extractions: it resolves the prior-cycle threads from `earnings-preview`, lands the editorial top story, and hands TelecomTV editors a commissioning-ready set of angles.

This skill is **editorial / synthesis**, not financial-research. It does **not** produce a DOCX report with price target, rating, DCF, or beat/miss tables for a stock. It does cite earnings numbers, AI capex disclosures, and headline KPIs — these are the evidence base for editorial claims about AI strategy, vendor ecosystem, and trajectory.

## When to Use

Use when the user requests:
- "Analyse [operator]'s Q3 earnings"
- "Write up [vendor]'s FY results"
- "Post-earnings coverage for [filer]"
- "[Filer] just reported — give me the editorial picture"
- "Cover [filer]'s earnings call from this morning"

**Do NOT use if:**
- The call hasn't happened yet → use `earnings-preview`
- The user wants extracted facts only (no editorial synthesis) → use the extraction skills directly (`ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker`, `earnings-call-themes`)
- The user wants a single tight angle → use `telecomtv-angle`
- The user wants a multi-cycle trajectory view of one filer → use `operator-trajectory-tracker`
- The user wants stock analysis / valuation / price target → this isn't the toolset

## Inputs

| Input | Required | Notes |
|---|---|---|
| Filer | Yes | Operator or vendor. Cross-check against the ANTA universe. |
| Reporting period | Yes | E.g. "Q3 2026", "FY 2025". The period just reported. |
| Source document(s) | Required for fresh extraction | Transcript URL/path + earnings-release URL/path + investor-deck (if available). If the doc(s) have already been ingested, the skill works from Supabase. |
| Audience hint | Optional | Default `analyst-editorial`. Same vocabulary as `telecomtv-angle`. |

## Output

A markdown editorial write-up (3–5 pages when rendered, ~1,500–3,000 words) plus a JSON sidecar capturing the synthesised state for downstream skills. The markdown is the deliverable; the JSON lets `telecomtv-angle`, `operator-trajectory-tracker`, and `editorial-leads` consume the synthesis.

### Markdown structure

```
# [Filer] [Reporting period] — Editorial Analysis

## Top story
[2–4 sentences. The single most editorially significant thing from this call. Specific, anchored to a verbatim quote or quantified disclosure.]

## Headlines from the print
[3–6 bullets. Reported numbers + AI capex + vendor moves. Each bullet anchored to a verbatim quote or table number with source ref.]

## Prior-cycle threads — resolved / extended / abandoned
[For each thread carried over from last cycle's `earnings-preview` `prior_cycle_threads`:
- **[Thread title]** — RESOLVED / EXTENDED / ABANDONED / UNADDRESSED
  - One sentence on what the call said.
  - Verbatim quote (≤500 chars) if applicable.
- ...]

## Editorial themes
[Pulled from `earnings-call-themes` for this transcript. List 3–6 themes with their tension flag. For tension=true themes, briefly characterise the tension.]

## AI deployment + capex
[Pulled from `ai-mentions-extractor` (deployment + contribution categories) + `ai-capex-tracker`. 1–3 paragraphs synthesising what the filer is actually doing on AI vs. what they previously committed to. Cite verbatim where impactful.]

## Vendor ecosystem moves
[Pulled from `vendor-mentions`. Notable new partnerships, expanded scope, vendors that have gone quiet. 1–2 paragraphs.]

## Notable absences
[Things the filer didn't mention that you'd have expected based on prior cycles, peer behaviour, or recent industry events. 2–4 bullets.]

## Commissioning-ready angles
[2–4 short bullets, each pointing at a specific TelecomTV piece an editor could commission today. Each cross-references back to the supporting evidence from above.]

## Connection to ANTA scoring
[1–2 sentences on what indicators this print provides evidence for, or how it might shift the filer's archetype / trajectory in the next ANTA cycle. Skip this section if the user explicitly says it's not relevant.]
```

### JSON sidecar

```json
{
  "source_doc": {
    "filer_kind": "operator",
    "filer_name": "VEON",
    "filer_operator_id": "<uuid>",
    "doc_type": "earnings_call_transcript",
    "title": "VEON Q3 2026 earnings call transcript",
    "source_url": "https://...",
    "filing_date": "2026-10-30",
    "reporting_period": "Q3 2026",
    "scoring_cycle_id": null,
    "reporting_currency": "USD",
    "extracted_by": "earnings-analysis v1"
  },
  "synthesis": {
    "top_story": "string — 2 to 4 sentences",
    "thread_resolutions": [
      {
        "thread_title": "string",
        "prior_cycle": "string e.g. 'Q2 2026'",
        "status": "resolved | extended | abandoned | unaddressed",
        "evidence_quote": "string ≤500 chars or null",
        "notes": "one sentence"
      }
    ],
    "commissioning_ready_angles": [
      {
        "angle_summary": "string — one sentence",
        "supporting_evidence_refs": ["string", ...],
        "suggested_skill_call": "telecomtv-angle | filing-diff | peer-set | none"
      }
    ],
    "anta_scoring_impact": "string — 1 to 2 sentences, or null if explicitly out of scope"
  }
}
```

## Workflow

### Step 1: Identify the filer + period

1. Confirm the filer; determine `filer_kind` and look up `filer_operator_id` from `anta-supabase`.
2. Confirm the just-reported `reporting_period`.
3. Determine `doc_type` (typically `'earnings_call_transcript'` but can be `'press_release'` if call hasn't happened yet, or `'investor_deck'` for material updates).
4. Populate the `source_doc` envelope.

### Step 2: Ensure structured extractions exist

Two paths:

**A. If the filing has already been ingested** (i.e. a `source_docs` row exists for the filer + reporting_period):
- Pull all child rows: `ai_mentions`, `vendor_mentions`, `ai_capex`, `earnings_call_themes`
- If any extraction is missing, run the corresponding skill now and ingest

**B. If the filing is fresh** (not yet in `anta-supabase`):
- Run all four extraction skills against the source documents:
  - `ai-mentions-extractor`
  - `vendor-mentions`
  - `ai-capex-tracker`
  - `earnings-call-themes`
- Each produces a `source_doc` envelope + child rows. Coordinate so all four point to the same `source_doc` record (loader UPSERTs by `(filer_name, reporting_period, doc_type)`).

The structured extractions are the evidence base for everything that follows. Don't skip this step — synthesise from extractions, not from re-reading the doc.

### Step 3: Pull prior-cycle threads

Query `anta-supabase` for the most recent `earnings-preview` output for this filer (if any) — specifically the `prior_cycle_threads` array. Each thread had a `resolves_if` condition. For each:
- Did the call meet the resolves_if condition? → status `resolved`
- Did the call address the thread but not resolve it? → status `extended` (or `abandoned` if the filer walked back)
- Did the call not address the thread at all? → status `unaddressed`

Capture verbatim quotes from the just-extracted `ai_mentions` / `ai_capex` / `earnings_call_themes` rows that anchor each status call.

### Step 4: Identify the top story

Pick **one** top story. Criteria:
- Anchored to a specific verbatim disclosure or quantified claim from this call
- Editorially significant — would TelecomTV want to lead with this?
- Specific — no generic "operator continues AI investment" framing

If the call was genuinely uneventful, the top story might be a notable absence ("X failed to update on Y after pressing in Q3"). That's a valid lead.

### Step 5: Write the synthesis sections

Write each section of the markdown structure. Rules:

- **Headlines from the print** — anchor every bullet to a verbatim quote or numeric disclosure. Cite source location (page or timestamp).
- **AI deployment + capex** — synthesise from `ai_mentions` (deployment + contribution categories) and `ai_capex` rows. Be honest about what's `spent` vs `committed` vs `aspirational`. Don't smooth over the disclosure-status distinction.
- **Vendor ecosystem moves** — synthesise from `vendor_mentions`. Highlight changes vs. prior cycle (vendors newly named, vendors no longer mentioned, scope changes on existing partnerships).
- **Notable absences** — pull from `earnings-call-themes` summary + cross-check against prior-cycle commitments. Be specific, not generic.
- **Commissioning-ready angles** — 2–4 bullets, each pointing at a specific TelecomTV piece. Each must cross-reference supporting evidence from earlier sections. Suggest the next skill to call (`telecomtv-angle` for full angle drafting, `filing-diff` for cycle-over-cycle comparison, `peer-set` for peer context).
- **Connection to ANTA scoring** — only include if the filer is in the ANTA universe AND the call has material implications for their archetype / trajectory / indicator scores. 1–2 sentences max.

### Step 6: Write the top story last

Counter-intuitively, write the "top story" section *after* drafting the rest — synthesis often surfaces the actual lead, which may differ from the most-pressed analyst topic on the call.

### Step 7: Build the JSON sidecar

Capture the synthesised state in the sidecar JSON for downstream consumption. The sidecar is what `editorial-leads` and `operator-trajectory-tracker` will pull from.

## Quality checklist

Before delivery:
- [ ] All four extraction skills run (or pulled from Supabase) before synthesis began
- [ ] Top story is specific, anchored to a verbatim quote or quantified disclosure
- [ ] Every "Headlines from the print" bullet has a source-anchored verbatim or numeric reference
- [ ] Every prior-cycle thread has a status (resolved / extended / abandoned / unaddressed) with evidence quote
- [ ] Disclosure status (spent / committed / announced / aspirational) preserved when discussing AI capex — not smoothed to "investing"
- [ ] Vendor ecosystem section explicitly notes changes vs. prior cycle (additions / removals / scope changes)
- [ ] Notable absences section is specific, not generic
- [ ] Commissioning-ready angles each name a specific filer + specific framing + suggested next skill
- [ ] Length: 1,500–3,000 words. Anything longer is wasted editorial time.
- [ ] No equity-research artefacts (no DCF, no price target, no rating, no beat/miss vs consensus framing — though specific number disclosure vs. prior-cycle guide is fine and editorially valuable)

## Important notes

- This skill assumes the four extraction skills land first. If the user invokes `/earnings` against a fresh transcript that hasn't been extracted, run all four first and ingest into Supabase before synthesising.
- The "Connection to ANTA scoring" section is the one place where the editorial layer touches the methodology layer. Be careful: the ANTA scoring conversation is a different one from editorial commissioning. Flag the connection but don't try to score.
- If the filer is a vendor (not an operator), the "ANTA scoring" section is typically not applicable — the ANTA Index scores operators, not vendors. Skip it for vendor filers unless you have specific reason to include it.
- For each commissioning-ready angle, suggest *one* downstream skill to call. Don't list 3 options — make a recommendation.

## Resources

### ../../references/source-doc-envelope.md
The shared `source_doc` envelope convention used across all telecom-analyst extraction skills.

## Dependencies

**Required:**
- `anta-supabase` MCP — for prior-cycle data, prior `earnings-preview` outputs, and post-extraction synthesis
- `ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker`, `earnings-call-themes` skills — these provide the evidence base; run before synthesis if not already in Supabase

**Optional:**
- `WebFetch` (URLs) / `Read` (local paths) — to fetch source documents if not already ingested
- `telecomtv-archive` MCP — to flag which angles TelecomTV has already covered (avoid duplicate commissioning recommendations)
- `peer-set` skill output — useful context if the analysis needs peer-comparison framing
