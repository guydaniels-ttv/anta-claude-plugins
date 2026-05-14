---
description: Produce a post-call editorial analysis for an operator or vendor's just-reported quarter — synthesises the structured extractions into a commissioning-ready editorial write-up
argument-hint: "[filer name] [reporting period e.g. Q3 2026] [transcript path or URL, optional if already ingested]"
---

# Earnings Analysis Command

Produce the post-call editorial layer on top of the structured extractions. Resolves prior-cycle threads, lands the editorial top story, surfaces commissioning-ready angles.

## Workflow

### Step 1: Parse the inputs

- **Filer name** — operator or vendor. Required. Cross-check against ANTA universe.
- **Reporting period** — required. E.g. "Q3 2026", "FY 2025".
- **Source documents** — required if the filing isn't already ingested. Transcript URL/path + optional earnings-release / investor-deck.
- **Audience hint** — optional; default `analyst-editorial`.

If the filer is missing, ask:
- "Which filer has just reported?"

If the source isn't ingested AND no source documents were provided, ask for the transcript URL/path.

### Step 2: Verify and load

1. Look up the filer in `anta-supabase` — canonical name + `filer_operator_id`.
2. Check if a `source_docs` row already exists for this filer + reporting_period + earnings_call_transcript. If yes, pull all child extractions.
3. If no existing extractions, fetch the source via `filings-store` MCP and prepare to run all four extraction skills.
4. Pull the most recent `/earnings-preview` output for this filer (if any) to get `prior_cycle_threads`.

### Step 3: Run extractions if needed

If extractions don't exist in Supabase yet:
- Run `ai-mentions-extractor` → ingest
- Run `vendor-mentions` → ingest
- Run `ai-capex-tracker` → ingest
- Run `earnings-call-themes` → ingest

All four should land against the same `source_docs` row (loader UPSERTs by `(filer_name, reporting_period, doc_type)`).

### Step 4: Invoke the skill

Use `skill: "earnings-analysis"` to do the synthesis layer on top of the extracted evidence base.

### Step 5: Deliver output

Provide:

1. **Markdown editorial write-up** — fixed structure: top story / headlines from the print / prior-cycle threads (resolved / extended / abandoned / unaddressed) / editorial themes / AI deployment + capex / vendor ecosystem moves / notable absences / commissioning-ready angles / connection to ANTA scoring.
2. **JSON sidecar** with the synthesised state — `source_doc` envelope + `synthesis` block — for downstream consumption by `editorial-leads`, `operator-trajectory-tracker`, etc.

### Step 6: Offer next steps

After delivering, offer (one or more, named specifically per the commissioning-ready angles in the output):
- "Want me to draft the full `/tv-angle` for [the strongest commissioning-ready angle]?"
- "Want me to run `/filing-diff` against this filer's previous cycle for the cycle-over-cycle picture?"
- "Want me to update `/peer-set` for this filer now that this cycle's data has landed?"
- "Want me to update [filer]'s entry in `operator-trajectory-tracker` with the new cycle?"

## Quality Checklist

Before delivery, confirm:
- [ ] Filer matched to ANTA canonical name; `filer_kind` correctly set; `filer_operator_id` populated for operator filers
- [ ] All four extraction skills ran (or pulled from Supabase) before synthesis
- [ ] Top story is specific, anchored to a verbatim quote or quantified disclosure
- [ ] Every "Headlines" bullet has a source-anchored verbatim or numeric reference
- [ ] Every prior-cycle thread has a status with supporting evidence quote
- [ ] Disclosure-status distinction (spent / committed / announced / aspirational) preserved
- [ ] Vendor ecosystem section explicit on changes vs. prior cycle
- [ ] Notable absences are specific, not generic
- [ ] Each commissioning-ready angle names a specific filer + framing + suggested next skill
- [ ] Length 1,500–3,000 words
- [ ] No equity-research artefacts (no DCF, no price target, no rating, no beat/miss vs consensus framing)
