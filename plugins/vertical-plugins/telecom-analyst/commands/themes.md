---
description: Extract the editorial themes from an earnings call transcript — what management led with, what analysts pressed, where the tension was, and what was conspicuously absent
argument-hint: "[path or URL to transcript] [filer name, optional] [reporting period, optional]"
---

# Earnings Call Themes Command

Read a single earnings call transcript and produce a themed editorial structure: prepared narrative vs. Q&A pressure, with explicit tension flags and notable absences.

## Workflow

### Step 1: Parse the inputs

- **Transcript** — path or URL to an earnings call transcript
- **Filer name** — optional; infer from the doc if not given. Determines `filer_kind` (operator vs. vendor) and `filer_operator_id` via `anta-supabase` lookup.
- **Reporting period** — optional; infer from the doc if not given (e.g. "Q3 2026"). Goes into `source_doc.reporting_period`.
- **ANTA scoring cycle** — optional; pass `scoring_cycle_id` if the run is tied to an ANTA cycle.

If the transcript is missing, ask:
- "Which transcript should I read? Paste a path or URL."

If the source isn't a transcript (e.g. a written filing), say so and stop — this skill only applies to call transcripts.

### Step 2: Verify and load

1. Fetch the transcript via `filings-store` MCP if a URL.
2. Confirm it is the latest cycle.
3. Look up the filer in `anta-supabase` for canonical name + `filer_operator_id`.
4. Pull prior-cycle themes for this filer if available — used for the absences check.

### Step 3: Invoke the skill

Use `skill: "earnings-call-themes"` to do the segmentation, theme clustering, scoring, and tension analysis.

### Step 4: Deliver output

Provide three things:

1. **Markdown summary table** sorted by `salience` desc then `share_of_call_pct` desc:

   | # | Theme | Category | Salience | Prepared | Q&A | Tension? | Mgmt quote (truncated) | Analyst quote (truncated) |
   |---|---|---|---|---|---|---|---|---|

2. **JSON object** matching the schema in the skill's `SKILL.md` — `source_doc` envelope + `themes` array + `summary` — ready for Supabase ingest.

3. **5–8 sentence narrative** for a TelecomTV editor. Lead with the dominant theme, name the most consequential tension, list notable absences. Don't be exhaustive — pick what an editor would actually want.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to ingest this into the ANTA Supabase `earnings_call_themes` table?" (Pending table design — flag if not yet available.)
- "Want me to draft `/tv-angle` pieces for the high-tension themes or notable absences?"
- "Want me to also run `/ai-mentions`, `/vendors`, and `/ai-capex` on the same transcript for the structured-data layer?"
- "Want me to compare these themes to the prior cycle via `/filing-diff --focus themes`?"

## Quality Checklist

Before delivery, confirm:
- [ ] Source is a transcript (not a filing)
- [ ] Filer matched to ANTA canonical name; `filer_kind` correctly set; `filer_operator_id` populated for operator filers
- [ ] Both prepared remarks and Q&A read in full
- [ ] 5–10 themes identified
- [ ] Tension flagged honestly — not every theme has tension
- [ ] At least one notable absence named in the summary
- [ ] All quotes verbatim, ≤500 chars
- [ ] Output JSON validates against the schema
