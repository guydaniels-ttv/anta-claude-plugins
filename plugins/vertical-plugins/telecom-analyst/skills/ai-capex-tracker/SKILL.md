---
name: ai-capex-tracker
description: Extract AI-related capex (and capex-adjacent) disclosures from an operator or vendor filing or earnings call. Capture every quantified spending claim that references AI / GenAI / ML / data centres / GPUs / AI infrastructure / network AI / customer-ops AI. Classify disclosure status (spent / committed / announced / aspirational), time horizon, AI scope, and whether the spend is actually capex or opex or ambiguous. Compute ratio to total capex where disclosed. Cross-reference prior-cycle disclosures for trajectory. Use when the user asks about AI capex, AI spending, AI investment, data centre capex, GPU spend, AI infra investment, AI investment commitments, or capex allocated to AI in a filing, transcript, annual report, 10-K, 20-F, quarterly, earnings call, or investor day.
---

# AI Capex Tracker

Extract every quantified AI-spending disclosure from a single source document. Designed for ANTA's tracking of how operators and vendors are actually putting money behind their AI rhetoric — and how that picture evolves cycle-over-cycle.

**The honest premise:** AI capex is almost never cleanly broken out in financial disclosure. This skill captures what *is* disclosed (numbers, ranges, ratios, commitments) and is honest about what isn't (bundled disclosures, aspirational targets without timelines, ambiguous capex/opex framing). It does not extrapolate or guess at numbers the filer hasn't given.

## When to Use

Use when the user requests:
- "Extract AI capex from [operator]'s Q3 results"
- "What is [vendor] disclosing on AI investment?"
- "Pull data-centre / GPU spending claims out of this 10-K"
- "How does [operator]'s AI capex compare to last quarter's commitment?"
- "AI investment numbers in this annual report"

**Do NOT use if:**
- The user wants qualitative AI commentary → use `ai-mentions-extractor`
- The user wants vendor partner mapping → use `vendor-mentions`
- The user wants total capex (not AI-specific) → this skill is scoped to AI; use a general capex skill if/when authored
- The doc contains zero AI capex disclosures → return an explicit "no quantified AI capex found in this doc," not a fabricated estimate

## Inputs

| Input | Required | Notes |
|---|---|---|
| Source document | Yes | Filing PDF/HTML, transcript, investor deck, press release. Path or URL — URL → `WebFetch`; local path → `Read`. |
| Filer (operator or vendor) | If not derivable | Cross-check ANTA universe. |
| Reporting period | If not derivable | E.g. Q3 2026, FY 2025. Goes into `source_doc.reporting_period`. |
| Doc type | If not derivable | Goes into `source_doc.doc_type` — see envelope reference. Required for UPSERT. |
| ANTA scoring cycle | Optional | Pass `scoring_cycle_id` UUID if the run is tied to an ANTA cycle. |
| Reporting currency | Required | The filer's reporting currency (ISO 4217). Goes into `source_doc.reporting_currency`. Used for normalisation; pulled from the doc if not given. |

## Output

JSON object with the shared `source_doc` envelope + `total_capex_disclosed` (optional) + `disclosures` array + `prior_cycle_comparison` array + summary narrative + a markdown summary table for human review. The envelope shape is shared with every other extraction skill — see [../../references/source-doc-envelope.md](../../references/source-doc-envelope.md) for the full convention.

The `total_capex_disclosed` block lives once at the top of the output for cleanliness; the loader denormalises it onto every `ai_capex` row's `total_capex_*` columns at insert time.

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
    "reporting_currency": "USD",
    "extracted_by": "ai-capex-tracker v1"
  },
  "total_capex_disclosed": {
    "amount": 350000000,
    "currency": "USD",
    "horizon": "Q3 2026",
    "horizon_type": "single-period",
    "quote": "Total capex of $350m in the quarter, slightly below our quarterly run-rate."
  },
  "disclosures": [
    {
      "id": 1,
      "amount_raw": "$50m",
      "amount_normalized": 50000000,
      "currency": "USD",
      "ratio_to_total_capex": 0.143,
      "ratio_source": "calculated",
      "time_horizon": "Q3 2026",
      "horizon_type": "single-period",
      "disclosure_status": "spent",
      "ai_scope": "AI infra",
      "capex_or_opex": "capex",
      "source_section": "CFO prepared remarks",
      "page_or_timestamp": "p. 8 / 22:15",
      "speaker": "CFO",
      "quote": "We deployed $50m of AI infrastructure capex this quarter, primarily GPU procurement for our customer-care GenAI rollout.",
      "vendors_involved": ["NVIDIA"],
      "context_tags": ["gpu", "customer-care", "genai"],
      "confidence": "high",
      "notes": "Clean disclosure — quantified, scoped, attributed to GPU spend; ratio to total capex computed from same-doc total."
    }
  ],
  "prior_cycle_comparison": [
    {
      "topic": "Three-year AI infra commitment",
      "prior_quote": "We plan to invest $500m in AI infrastructure over the next three years.",
      "prior_cycle": "Q2 2026",
      "current_quote": "We are now committing $750m over four years to our AI infrastructure programme.",
      "delta": "Commitment scaled up by 50% in size, extended by one year. Implied annual run-rate up from ~$167m to ~$187m."
    }
  ],
  "summary": "One quantified spent disclosure ($50m AI infra capex in Q3 2026, 14% of total capex). The three-year AI infra commitment was revised upward to $750m over four years (vs $500m/3y previously). No GenAI-specific commitments quantified yet."
}
```

## Disclosure Status Taxonomy

Four statuses. This is the most important classification axis — different statuses have very different signal value.

| Status | What it means | Tense / signal |
|---|---|---|
| **spent** | Past, realised, hit the income statement / cash flow already. | Past tense: "we spent," "we deployed," "incurred," "of which AI represented". |
| **committed** | Contractually committed — signed contracts, multi-year purchase agreements, binding orders. | "we have committed," "under contract," "signed agreements totalling," "POs in place for". |
| **announced** | Publicly announced plan with a stated horizon. Not yet contractually bound, but has shape. | "we plan to invest," "our programme will deploy," "by FY28 we will have invested". |
| **aspirational** | Vague target, no plan, no timeline, or so loosely scoped as to be promotional. | "we expect to invest significantly," "AI is a major investment area," "meaningful capex". |

See [references/disclosure-status.md](references/disclosure-status.md) for worked examples, especially `committed` vs `announced` (the most common judgement call), and how to handle hedged language like "up to" and "in the range of."

## AI Scope Taxonomy

What the spend is *for*. One per disclosure.

| Scope | Definition |
|---|---|
| **generic AI** | "Investing in AI," no further specificity. |
| **GenAI** | Generative AI specifically — foundation models, copilots, chatbots. |
| **AI infra** | GPUs, data-centre buildout for AI, AI silicon, AI compute capacity. |
| **network AI** | AI in the RAN, Core, transport — anomaly detection, energy optimisation, intent-based networking. |
| **customer-ops AI** | Customer-care, self-service, agent assist, personalisation. |
| **field-ops AI** | Network ops, field force, predictive maintenance, dispatch optimisation. |
| **fraud / risk AI** | Fraud detection, AML, churn prediction, risk scoring. |
| **AI talent / hiring** | Hiring, training, skill build — often opex, not capex. Flag in `capex_or_opex`. |
| **specific use case** | Named system or platform. Capture the name in `context_tags`. |

## Capex vs Opex

Many "AI investment" disclosures are ambiguous as to capex/opex. Be honest.

| Value | When |
|---|---|
| **capex** | Explicitly capex — appears in capex section, capex bridge, capital expenditure narrative. Or named items that are clearly capex (data-centre buildout, GPU purchase, network gear). |
| **opex** | Explicitly opex — running costs, cloud consumption, third-party services, hiring. Or appears in opex / cost-base narrative. |
| **mixed** | Disclosure explicitly combines both ("we are investing in AI through capex and opex"). |
| **unclear** | The word "investment" is used without disambiguation. **Default to this** when in doubt — don't pretend the disclosure is clean when it isn't. |

## Trigger Patterns

When scanning, look for AI keyword + number proximity. Patterns:

- `[amount]` near `AI | GenAI | ML | artificial intelligence | data centre | data center | GPU | AI infra | AI infrastructure | machine learning`
- `[X]% of our capex` near AI keyword
- `capex allocated to AI`
- `AI capex` / `AI investment` / `AI spend` / `AI spending`
- `GPU procurement` / `GPU capacity` / `GPU [number]`
- `data centre buildout` / `data center buildout` / `compute capacity`
- `[over the next N years] [amount] [in / for / on] AI`

For each match, expand to the **full sentence plus the sentence before and after** for context.

## Workflow

### Step 1: Identify and verify

1. Confirm doc is the latest cycle (filing_date captured).
2. Identify the filer; determine `filer_kind` (operator or vendor) and look up the canonical name + `filer_operator_id` (operator filers only) from `anta-supabase`.
3. Determine `doc_type` from the 12-value enum (annual_report / 10-K / 20-F / quarterly_report / earnings_call_transcript / investor_day / press_release / investor_deck / analyst_day / cmd / regulatory_filing / other). Required for UPSERT.
4. Capture `reporting_period`, `reporting_currency`, and `scoring_cycle_id` (if the run is bound to an ANTA cycle). Populate the full `source_doc` envelope before extracting disclosures.

### Step 2: Locate capex sections

Expect AI capex content in:

**Filings:**
- Capex / capital expenditure section
- Capex bridge or reconciliation
- MD&A — capital allocation narrative
- Notes on commitments and contractual obligations

**Transcripts:**
- CFO prepared remarks (highest density)
- CEO prepared remarks — strategic commitments
- Analyst Q&A — where vague disclosures get pressed for specifics

**Press releases / investor decks:**
- Capex slide, capital allocation slide
- AI strategy slide with investment commitments

### Step 3: Extract total capex (if disclosed)

If the doc discloses total capex for the same period, capture it in `total_capex_disclosed`. Used downstream to compute ratios.

If not disclosed, leave `total_capex_disclosed: null`. **Don't pull total capex from outside the doc** — that's a different skill (KPI extract).

### Step 4: Extract each AI capex disclosure

For each AI keyword + number match:
1. Capture verbatim quote (≤500 chars)
2. Parse `amount_raw` and normalize to integer in `amount_normalized` (handle €/$/£/¥ + m/bn/cr suffixes)
3. Set `currency`
4. Determine `time_horizon` (the period the number applies to)
5. Set `horizon_type`: `single-period`, `annual-rate`, `multi-year-cumulative`, `by-date`, `range`
6. Classify `disclosure_status` (per taxonomy above)
7. Classify `ai_scope`
8. Determine `capex_or_opex` (default to `unclear` when truly ambiguous)
9. Note `source_section`
10. Capture `vendors_involved` (named recipients of the spend — cross-check the ANTA vendor universe)
11. Capture `context_tags` (lowercase short tags)
12. Compute `ratio_to_total_capex` if both this disclosure and `total_capex_disclosed` are in the same currency and horizon; set `ratio_source: "calculated"` or `"disclosed"` accordingly

### Step 5: Prior-cycle comparison

Query `anta-supabase` for prior `ai_capex` disclosures for this filer. For each prior commitment that has a current-cycle counterpart, populate `prior_cycle_comparison`:
- Note revisions (size, timeline, scope changes)
- Note commitments confirmed as spent
- Note commitments that have gone quiet (no current-cycle equivalent)

The "gone quiet" case is editorially valuable — but defer the comprehensive removed-list to `filing-diff --focus capex`.

### Step 6: Summary

3–6 sentence narrative for an editor. Lead with the most newsworthy disclosure (usually the largest `spent` or `committed` number, or a material revision). State explicitly if there are zero quantified disclosures — that's news too.

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
    "reporting_currency": "string (ISO 4217)",
    "extracted_by": "ai-capex-tracker v1"
  },
  "total_capex_disclosed": {
    "amount": "number",
    "currency": "string",
    "horizon": "string",
    "horizon_type": "single-period | annual-rate | multi-year-cumulative | by-date",
    "quote": "string ≤500 chars"
  },
  "disclosures": [
    {
      "id": "integer",
      "amount_raw": "string — as it appeared in the doc, e.g. '$50m' or 'INR 500 cr'",
      "amount_normalized": "number — integer in the major unit of `currency`",
      "currency": "string (ISO 4217)",
      "ratio_to_total_capex": "number 0..1 or null",
      "ratio_source": "calculated | disclosed | null",
      "time_horizon": "string",
      "horizon_type": "single-period | annual-rate | multi-year-cumulative | by-date | range",
      "disclosure_status": "spent | committed | announced | aspirational",
      "ai_scope": "generic AI | GenAI | AI infra | network AI | customer-ops AI | field-ops AI | fraud / risk AI | AI talent / hiring | specific use case",
      "capex_or_opex": "capex | opex | mixed | unclear",
      "source_section": "string",
      "page_or_timestamp": "string",
      "speaker": "string or null",
      "quote": "string ≤500 chars, verbatim",
      "vendors_involved": ["string", ...],
      "context_tags": ["string", ...],
      "confidence": "high | medium | low",
      "notes": "string — one sentence"
    }
  ],
  "prior_cycle_comparison": [
    {
      "topic": "string",
      "prior_quote": "string",
      "prior_cycle": "string",
      "current_quote": "string or null",
      "delta": "string"
    }
  ],
  "summary": "string — 3 to 6 sentences"
}
```

## Quality Checklist

Before delivery:
- [ ] `source_doc` envelope is complete and valid (filer_kind set, filer_name canonical, doc_type set, reporting_period set, reporting_currency set)
- [ ] `filer_operator_id` populated only when `filer_kind == 'operator'` AND a match exists; else null
- [ ] Source doc verified as latest cycle (filing_date captured)
- [ ] `total_capex_disclosed` populated only if explicitly in the doc (no external pulls)
- [ ] Every AI-keyword + number proximity scanned, including in lists and footnotes
- [ ] `amount_normalized` arithmetic correct for m/bn/cr/lakh suffixes
- [ ] `disclosure_status` honest — vague targets are `aspirational`, not `announced`
- [ ] `capex_or_opex` defaults to `unclear` when truly ambiguous; not silently coerced to `capex`
- [ ] `vendors_involved` populated and cross-checked against ANTA vendor universe
- [ ] `ratio_to_total_capex` only computed when currencies and horizons match
- [ ] Prior-cycle comparison done where data is in Supabase
- [ ] If zero quantified disclosures found, that is stated explicitly in the summary
- [ ] No external numbers introduced — only what's in the doc

## Resources

### references/disclosure-status.md
Worked examples for the four statuses, especially `committed` vs `announced` (the most common call), and how to handle hedged language ("up to," "in the range of," "around").

### ../../references/source-doc-envelope.md
The shared `source_doc` envelope convention used across all telecom-analyst extraction skills.

## Dependencies

**Required:**
- `WebFetch` (URLs) / `Read` (local paths) — to fetch source documents
- `anta-supabase` MCP — to look up canonical filer name, vendor universe, and prior-cycle AI capex disclosures

**Optional:**
- `ai-mentions-extractor` skill — every AI capex disclosure is also an AI mention (in the `contribution` or `strategy` category, depending on `disclosure_status`); run both when the user wants both lenses
- `filing-diff --focus capex` — for the comprehensive removed-list across cycles
