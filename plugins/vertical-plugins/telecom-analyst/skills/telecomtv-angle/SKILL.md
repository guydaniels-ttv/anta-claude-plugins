---
name: telecomtv-angle
description: Draft editorial story angles from a single primary-source document (filing, transcript, investor day, press release) suitable for TelecomTV coverage. Each angle includes a hook, an audience, the supporting evidence (with verbatim quotes), an estimate of novelty vs. prior coverage in the TelecomTV archive, and a recommended length and format. Use when the user asks for editorial angles / story ideas / TelecomTV pitches / what's the story / what's worth covering / what an editor should write about from a filing or transcript.
---

# TelecomTV Angle

Generate editorial story angles from a single primary-source document — filing, earnings call transcript, investor day, press release, or analyst-day deck. Each angle is a candidate for TelecomTV coverage with enough scaffolding (hook, evidence, recommended format) for an editor to commission.

This skill is **editorial**, not extractive. It assumes the structured-data layer (mentions, vendors, capex, themes) is either already extracted or will be alongside. Its job is to ask: "if I were writing about this filing for a telecoms-industry audience, what would I write?" — and to be honest about which angles are genuinely fresh vs. which are restatements of what TelecomTV has already covered.

## When to Use

Use when the user requests:
- "What are the editorial angles for [operator]'s Q3 results?"
- "Pitch me TelecomTV stories from this transcript"
- "What should we cover from this investor day?"
- "Draft angles for a TelecomTV editor from this filing"
- "What's the story here?"

**Do NOT use if:**
- The user wants raw extraction → use `ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker`, or `earnings-call-themes`
- The user wants a finished article → this skill produces angle drafts, not articles. Drafting is a separate downstream task.
- The user wants market commentary not anchored in a specific document → angles must trace to a source doc

## Inputs

| Input | Required | Notes |
|---|---|---|
| Source document | Yes | Filing PDF/HTML, transcript, investor deck, press release. Path or URL. |
| Filer name | If not derivable | Cross-check ANTA universe. |
| Reporting period | If not derivable | E.g. "Q3 2026". Goes into `source_doc.reporting_period`. |
| Doc type | If not derivable | Goes into `source_doc.doc_type`. Required for UPSERT. |
| ANTA scoring cycle | Optional | Pass `scoring_cycle_id` if the run is tied to an ANTA cycle. |
| Audience hint | Optional | Default is `analyst-editorial` (TelecomTV's primary audience). Other values: `cxo` (operator C-suite readership), `vendor-strategy` (vendor strategists), `regulatory` (policy desks). |

## Output

JSON object with the shared `source_doc` envelope + an `angles` array + a summary narrative + a markdown summary table for human review. The envelope shape is shared with every other extraction skill — see [../../references/source-doc-envelope.md](../../references/source-doc-envelope.md) for the full convention.

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
    "extracted_by": "telecomtv-angle v1"
  },
  "angles": [
    {
      "id": 1,
      "headline_draft": "VEON's Pakistan GenAI deflection hits 35% — but the CSAT question lingers",
      "angle_type": "tension-feature",
      "audience": "analyst-editorial",
      "hook": "VEON disclosed a 35% deflection rate from its Pakistan customer-care GenAI agent — the strongest concrete deployment metric from any operator in the cycle so far. Two analysts immediately pressed on whether the deflection was real demand reduction or customers giving up. Management acknowledged 'modest' CSAT pressure but didn't quantify, leaving open the question every operator deploying customer-ops GenAI now has to answer.",
      "supporting_evidence": [
        {
          "source": "transcript Q3 2026, 37:22",
          "quote": "We've rolled out our self-care GenAI agent in Pakistan, handling 60% of inbound tier-1 queries with a 35% deflection rate from human agents.",
          "speaker": "CEO Kaan Terzioğlu"
        },
        {
          "source": "transcript Q3 2026, 1:12:08",
          "quote": "On the Pakistan GenAI agent — what's the CSAT impact looking like, and how are you measuring whether the deflection is real demand reduction vs. customers giving up?",
          "speaker": "Analyst — Goldman Sachs"
        }
      ],
      "novelty": "fresh",
      "freshness_basis": "TelecomTV archive search returned no prior coverage of operator-level GenAI deflection rates being pressed for CSAT impact. The 35% number itself is new disclosure.",
      "recommended_length": "medium",
      "recommended_format": "feature",
      "competitive_context": "Vodafone disclosed a 22% deflection on its UK GenAI rollout in Q1; Orange has not yet disclosed a deflection metric. VEON's number is the highest disclosed in the sector to date.",
      "risks": [
        "CSAT data may emerge later that materially changes the framing — feature should hedge on this point",
        "The 35% may be cycle-noisy; check whether VEON has disclosed a quarter-over-quarter trajectory"
      ],
      "tags": ["genai", "customer-care", "deflection", "csat", "pakistan", "veon"],
      "confidence": "high",
      "notes": "Strong angle — quantified, contested, fresh, peer-comparable."
    }
  ],
  "summary": "Three angles, dominated by the Pakistan GenAI tension. The capex-mix angle (GPU spend creeping up while management holds the headline number) is the most analytically rich for an editorial-analyst audience but requires reading two filings to land. The tower-co absence angle is genuinely interesting but soft — recommend covering only if a peer transaction lands in the next two weeks."
}
```

## Angle Types

Pick the strongest fit per angle. Different types imply different editorial treatment downstream.

| Type | What it is |
|---|---|
| **tension-feature** | A framing-vs-pressure tension from the doc. Usually high editorial value. |
| **disclosure-news** | A specific new disclosure (number, partnership, deployment, exit) worth a short news piece. |
| **trajectory-analysis** | Cycle-over-cycle movement on a metric or commitment — requires prior-cycle context. |
| **absence-analysis** | Notable absences — what management didn't say. Requires peer or prior-cycle context. |
| **peer-contrast** | This filer's position vs. named peers on the same question. Requires peer-set context. |
| **policy-implication** | The disclosure has regulatory or policy resonance worth flagging. |
| **vendor-ecosystem** | Story is about the vendor mix or partnership reshuffle, not the operator's own actions. |
| **cxo-takeaway** | Practitioner angle — operational lesson for other operators / vendor strategists. |

## Audience

Default to `analyst-editorial` (TelecomTV's primary audience: telecoms analysts, sector journalists, industry editors). Use other values when the angle clearly skews:

- **analyst-editorial** — telecoms analysts, sector journalists; default
- **cxo** — operator C-suite readership; angle has direct operational lesson
- **vendor-strategy** — vendor strategists planning competitive positioning
- **regulatory** — policy desks, regulators, public-sector readers

## Novelty

Honest assessment of how novel the angle is.

| Value | Definition |
|---|---|
| **fresh** | TelecomTV has not covered this angle on this filer or comparable filers recently. The disclosure or framing is new. |
| **incremental** | TelecomTV has covered the broader topic, but this filing adds a meaningful new data point or development. |
| **restatement** | The disclosure repeats prior coverage with no material new data. Lower editorial priority — only worth a short note. |
| **unknown** | Could not check the TelecomTV archive (e.g. archive MCP unavailable). Flag explicitly so the editor knows to check. |

Always populate `freshness_basis` (one or two sentences) explaining what was checked and what was found. If `novelty: "unknown"`, say so explicitly — don't fabricate a freshness check.

## Recommended Length & Format

`recommended_length`: `"short"` (~250 words, news), `"medium"` (~600 words, feature), `"long"` (~1,200 words, analysis).

`recommended_format`: `"news"` (factual report), `"feature"` (narrative around a tension or trajectory), `"analysis"` (multi-source argumentative piece), `"column"` (opinion-led, named author voice).

These are recommendations only — the editor decides.

## Workflow

### Step 1: Identify and verify the source

1. Confirm the source doc is current (filing_date captured).
2. Identify the filer; determine `filer_kind` and look up `filer_operator_id` from `anta-supabase` for operator filers.
3. Determine `doc_type` from the 12-value enum. Capture `reporting_period` and `scoring_cycle_id` (if known). Populate the `source_doc` envelope.

### Step 2: Build the evidence layer

Two paths, in order of preference:

1. **If `ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker`, and/or `earnings-call-themes` have already been run** on this doc and their output is available, use it. Don't re-extract. Reference back to specific extracted records by their content.

2. **Otherwise** read the doc end-to-end with editorial intent: what would a sector journalist underline? Capture verbatim quotes for the candidate angles you spot.

### Step 3: Cluster into angle candidates

Aim for 2–5 angles per doc. Fewer if the doc is genuinely thin; more only if the doc had multiple substantive disclosures.

For each candidate angle:
- What's the **single sentence** that captures the story?
- What's the **angle type** from the rubric?
- What's the **evidence** (verbatim quotes + source references)?
- Who is the **audience**?

### Step 4: Check freshness

If `telecomtv-archive` MCP is available, query it for prior coverage of:
- This filer + this topic
- The same topic on comparable filers in the same cycle

Set `novelty` and write `freshness_basis`. If the archive MCP is unavailable, set `novelty: "unknown"` and say so explicitly.

### Step 5: Pull peer / competitive context

If `anta-supabase` is available, pull comparable disclosures from peer operators on the same topic in the same or recent cycles. Use this to populate `competitive_context` (one or two sentences). If no peer context is available, omit the field — don't fabricate.

### Step 6: Write the hook + risks

For each angle:
- Write the **hook** — a paragraph (2–4 sentences) that an editor could use as the lead. Specific, anchored, no boilerplate.
- Write the **headline draft** — short, specific, no clickbait. The editor will rewrite — your job is to give them a starting point.
- Write **risks** — 1–3 specific things that could undermine the piece if they emerge later (data revisions, missing context, peer transactions about to land).
- Set **confidence** on the angle holding up under scrutiny.

### Step 7: Summary

3–5 sentence summary for the commissioning editor. Lead with the strongest angle. Note which angles are most newsworthy now vs. which would benefit from waiting for adjacent context.

## Output Schema

```json
{
  "source_doc": {
    "filer_kind": "operator | vendor",
    "filer_name": "string — canonical name from ANTA universe",
    "filer_operator_id": "uuid or null",
    "doc_type": "annual_report | 10-K | 20-F | quarterly_report | earnings_call_transcript | investor_day | press_release | investor_deck | analyst_day | cmd | regulatory_filing | other",
    "title": "string — human-readable doc title",
    "source_url": "string or null",
    "filing_date": "YYYY-MM-DD or null",
    "reporting_period": "string e.g. 'Q3 2026'",
    "scoring_cycle_id": "uuid or null",
    "reporting_currency": null,
    "extracted_by": "telecomtv-angle v1"
  },
  "angles": [
    {
      "id": "integer",
      "headline_draft": "string ≤120 chars",
      "angle_type": "tension-feature | disclosure-news | trajectory-analysis | absence-analysis | peer-contrast | policy-implication | vendor-ecosystem | cxo-takeaway",
      "audience": "analyst-editorial | cxo | vendor-strategy | regulatory",
      "hook": "string — 2 to 4 sentences, the editorial lead",
      "supporting_evidence": [
        {
          "source": "string — short reference, e.g. 'transcript Q3 2026, 37:22' or '10-K p.42'",
          "quote": "string ≤500 chars, verbatim",
          "speaker": "string or null"
        }
      ],
      "novelty": "fresh | incremental | restatement | unknown",
      "freshness_basis": "string — one or two sentences on what was checked",
      "recommended_length": "short | medium | long",
      "recommended_format": "news | feature | analysis | column",
      "competitive_context": "string or null — one or two sentences on peer / sector context",
      "risks": ["string", ...],
      "tags": ["string", ...],
      "confidence": "high | medium | low",
      "notes": "string — one sentence"
    }
  ],
  "summary": "string — 3 to 5 sentences"
}
```

## Quality Checklist

Before delivery:
- [ ] `source_doc` envelope is complete and valid
- [ ] `filer_operator_id` populated only when `filer_kind == 'operator'` AND a match exists; else null
- [ ] 2–5 angles drafted (not fewer, not more)
- [ ] Every angle has `supporting_evidence` with at least one verbatim quote (≤500 chars) + source reference
- [ ] `novelty` honestly assessed; if `"unknown"` because archive MCP unavailable, said so explicitly
- [ ] `freshness_basis` populated for every angle (even if `"unknown"`)
- [ ] `competitive_context` is genuine (from `anta-supabase`) or omitted — never fabricated
- [ ] `headline_draft` is specific and not clickbait
- [ ] `risks` lists at least one specific failure mode per angle (or empty array if genuinely none)
- [ ] No paraphrased quotes in `supporting_evidence`
- [ ] Output JSON is valid and schema-conformant

## Resources

### ../../references/source-doc-envelope.md
The shared `source_doc` envelope convention used across all telecom-analyst extraction skills.

## Dependencies

**Required:**
- `filings-store` MCP — to fetch the source document
- `anta-supabase` MCP — for canonical filer lookup, peer-set context, and prior-cycle disclosure context

**Optional:**
- `telecomtv-archive` MCP — for the freshness check; `novelty` defaults to `"unknown"` if unavailable
- `earnings-call-themes`, `ai-mentions-extractor`, `vendor-mentions`, `ai-capex-tracker` — these produce the structured evidence layer this skill draws on; if available in the conversation, prefer them over re-reading the doc
