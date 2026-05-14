---
name: filing-diff
description: Compare an operator or vendor's current-cycle filing or earnings call against the prior-cycle counterpart and surface what's new, what's gone (notable absences), what's reworded, and what's numerically changed. Optionally narrow the comparison to a single focus area — AI mentions, risk factors, guidance, capex, strategy, vendor partnerships. Use when the user asks to diff, compare, contrast, or check quarter-over-quarter / cycle-over-cycle changes in filings, transcripts, 10-Ks, 20-Fs, quarterlies, annual reports, or investor day decks. Output is a structured change list ranked by editorial value, designed for TelecomTV editorial triage.
---

# Filing Diff

Semantic comparison of two cycles of the same operator or vendor's primary-source docs. Produces a ranked list of **materially interesting** changes — not a textual diff.

Editorially, the most valuable bucket is often **what's been removed**: things management was emphasising last quarter that have gone quiet. This skill is built to surface those, not bury them.

## When to Use

Use when the user requests:
- "Diff [operator]'s Q3 vs Q2 transcripts"
- "What's changed in [vendor]'s annual report compared to last year?"
- "Compare AI mentions in [operator]'s latest filing to the prior cycle"
- "What's new / gone / reworded in this quarter's 20-F?"
- "Risk factors diff for [vendor], FY26 vs FY25"

**Do NOT use if:**
- The user wants a fresh extraction of one cycle only → use `ai-mentions-extractor`, `operator-kpi-extract`, etc.
- The user wants a long-form thesis update → use `thesis-tracker`
- Both docs are missing → ask for them first
- The two docs are different document types (e.g. annual report vs press release) — these aren't comparable like-for-like. Tell the user and suggest the closest pair.

## Inputs

| Input | Required | Notes |
|---|---|---|
| Current-cycle source doc | Yes | Path or URL. The "new" side of the diff. |
| Prior-cycle source doc | If not findable in `anta-supabase` | The "old" side. Skill should auto-locate from prior-cycle context in Supabase before asking; if a fresh fetch is needed, use `WebFetch` / `Read`. |
| Operator/vendor name | If not derivable from the docs | Cross-check against ANTA universe. |
| Focus | Optional | `all` (default), `ai`, `risks`, `guidance`, `capex`, `strategy`, `vendors`. Narrows the comparison. |

The two docs should be the same type (transcript vs transcript, annual vs annual). If only one type is available for the prior cycle, flag it and proceed with the caveat noted in `notes`.

## Output

**Primary deliverable**: a JSON object with a `changes` array, each change classified and ranked. Plus a markdown summary table and a 3–6 sentence editorial narrative.

```json
{
  "operator_or_vendor": "VEON",
  "current_cycle": "Q3 2026",
  "prior_cycle": "Q2 2026",
  "current_doc": "Q3 2026 earnings call transcript",
  "prior_doc": "Q2 2026 earnings call transcript",
  "focus": "all",
  "changes": [
    {
      "id": 1,
      "topic": "AI-driven personalisation contribution",
      "section": "CFO prepared remarks",
      "status": "new",
      "current_quote": "AI-driven personalisation contributed 3pp to ARPU growth this quarter.",
      "prior_quote": null,
      "delta_summary": "First quantified claim of AI revenue contribution.",
      "editorial_value": "high",
      "tags": ["ai", "contribution", "arpu"]
    },
    {
      "id": 2,
      "topic": "5G FWA expansion in Ukraine",
      "section": "MD&A",
      "status": "removed",
      "current_quote": null,
      "prior_quote": "We continue to expand 5G FWA across Ukraine despite ongoing challenges.",
      "delta_summary": "No mention this cycle. Notable absence — was a featured strategic priority last quarter.",
      "editorial_value": "medium",
      "tags": ["strategy", "removed"]
    }
  ],
  "summary": "Three editorially significant changes. The first quantification of AI revenue contribution is the headline. CEO doubled down on Pakistan GenAI deployment with new deflection numbers. Ukraine FWA messaging went quiet — worth a follow-up question."
}
```

## Change Statuses

Five statuses. Pick the strongest fit.

| Status | Definition |
|---|---|
| **new** | Topic present this cycle, absent in prior cycle. |
| **removed** | Topic present in prior cycle, absent this cycle. Often the most editorially interesting. |
| **reworded** | Same topic, materially different language but the substance is unchanged. E.g. "we are deploying" → "we have deployed." Capture both quotes. |
| **materially-changed** | Same topic, substantive change — number moved, strategy pivoted, partner swapped, scope expanded/narrowed. |
| **unchanged** | Substantively identical to prior cycle. **Omit from output** unless `focus` is narrow enough that omitting would leave the report empty. Boilerplate that recurs verbatim is `unchanged` and not interesting. |

**Recycled boilerplate**: if the same hype sentence appears verbatim in both cycles (especially common in marketing-heavy CEO remarks), classify as `unchanged` and omit. Don't pad the output.

See [references/comparison-rules.md](references/comparison-rules.md) for worked examples on each status, plus the `reworded` vs `materially-changed` distinction (which is the most common judgement call).

## Editorial Value

Three levels. Rate by what a TelecomTV editor would actually run a story or ask a follow-up question on.

| Level | When | Examples |
|---|---|---|
| **high** | Genuinely new information, quantified, attributable. Or a notable absence that suggests a strategic shift. | First quantified AI contribution. A previously named partner has disappeared. Guidance lowered. |
| **medium** | A meaningful change but expected or partly trailed previously. | Deflection rate moved. Capex split clarified. New vendor named for a previously-announced initiative. |
| **low** | Real change but probably only relevant to a model-builder or analyst, not an editor. | Tax-rate guidance moved. Minor segment reclassification. |

If you can't decide between `high` and `medium`, pick `medium`. Don't inflate.

## Workflow

### Step 1: Identify both docs

1. Confirm the current-cycle doc is the latest (filing date verified).
2. Find the prior-cycle counterpart:
   - First, check `anta-supabase` for a prior structured extraction (e.g. prior `ai_mentions` rows). If present, prefer it — you avoid re-extracting.
   - Else, fetch the prior doc via `WebFetch` (URL) or `Read` (local path).
   - Else, ask the user.
3. Verify both docs are the same type. If not, flag and proceed with caveat.

### Step 2: Build topic lists for both docs

For each doc, build a structured list of topics covered, scoped to `focus`:

- For `focus: all` — major topics across prepared remarks / MD&A / risk factors / guidance / strategy / capex / partnerships / named systems
- For `focus: ai` — call `ai-mentions-extractor` on each doc (or use prior extractions from Supabase if available)
- For `focus: risks` — extract risk-factor items
- For `focus: guidance` — extract numerical guidance ranges
- For `focus: capex` — extract capex disclosures
- For `focus: strategy` — extract strategy statements (forward-looking intent)
- For `focus: vendors` — extract named third-party partners

### Step 3: Pair topics across cycles

Match topics semantically, not textually. Two quotes about "AI in customer care in Pakistan" match even if worded very differently. Use the surrounding context (system named, market named, function named) to disambiguate.

Unpaired current-cycle topics → `new`.
Unpaired prior-cycle topics → `removed`.
Paired topics → step 4.

### Step 4: Classify paired topics

For each paired topic, classify as `unchanged`, `reworded`, or `materially-changed`:

- Same substance, same numbers, similar language → **unchanged** (omit from output)
- Same substance, same numbers, materially different language → **reworded**
- Different number, different scope, different partner, different commitment → **materially-changed**

When in doubt between reworded and materially-changed, go with **materially-changed** — under-classification of substantive change is worse than over-classification.

### Step 5: Rate editorial value, summarise

For each change in the output (everything except `unchanged`):
- Rate editorial value
- Write a one-line `delta_summary`: the substance of what changed, not just that something changed
- Add `tags`: lowercase short tags including the section type (e.g. `risk`, `guidance`, `strategy`, `ai`, `capex`) and the status (`new`, `removed`, etc.) and any vendor names

Sort the output `changes` array by editorial_value (high → low) then by status priority (removed > new > materially-changed > reworded).

Emit the 3–6 sentence editorial summary: what an editor needs to know in 30 seconds. Lead with the single most consequential change.

## Output Schema

```json
{
  "operator_or_vendor": "string",
  "current_cycle": "string",
  "prior_cycle": "string",
  "current_doc": "string",
  "prior_doc": "string",
  "focus": "all | ai | risks | guidance | capex | strategy | vendors",
  "changes": [
    {
      "id": "integer",
      "topic": "string — short noun phrase",
      "section": "string — e.g. 'CFO prepared remarks', 'MD&A', 'Risk factors'",
      "status": "new | removed | reworded | materially-changed",
      "current_quote": "string ≤500 chars or null",
      "prior_quote": "string ≤500 chars or null",
      "delta_summary": "string — one line, substance not meta",
      "editorial_value": "high | medium | low",
      "tags": ["string", ...]
    }
  ],
  "summary": "string — 3 to 6 sentences"
}
```

## Quality Checklist

Before delivery:
- [ ] Both docs verified as same type and correct cycles
- [ ] Operator/vendor matched to ANTA canonical name
- [ ] `removed` changes are not silently dropped — they're often the most valuable
- [ ] No `unchanged` rows in the output (unless `focus` is so narrow that omitting them empties the report)
- [ ] Each `delta_summary` describes the substance, not just "this changed"
- [ ] `editorial_value` is honestly assessed — not everything is `high`
- [ ] Changes are sorted by editorial value, then status priority
- [ ] If a prior structured extraction was reused from Supabase, that's noted in the summary

## Resources

### references/comparison-rules.md
Worked examples for each status, especially `reworded` vs `materially-changed` (the most common judgement call), and how to assess editorial value.

## Dependencies

**Required:**
- `WebFetch` (URLs) / `Read` (local paths) — to fetch source documents
- `anta-supabase` MCP — to find prior-cycle docs and to reuse prior structured extractions (e.g. `ai_mentions` rows) where available

**Optional:**
- `ai-mentions-extractor` skill — called internally when `focus: ai`
- `telecomtv-archive` MCP — for editorial context on prior coverage
