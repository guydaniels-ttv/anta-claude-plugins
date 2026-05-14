---
description: Build a pre-earnings editorial preview for an operator or vendor — what to listen for on the call, which prior-cycle threads need resolution, anchored in ANTA Supabase prior-cycle data
argument-hint: "[filer name] [reporting period e.g. Q4 2026, optional]"
---

# Earnings Preview Command

Generate a focused pre-earnings editorial preview for a single filer. Output is anchored in their last cycle's disclosures and the open editorial threads those disclosures created — designed for TelecomTV pre-positioning + ANTA evidence anticipation.

## Workflow

### Step 1: Parse the inputs

- **Filer name** — operator or vendor. Required. Cross-check against the ANTA universe.
- **Reporting period** — optional; infer the next period from `source_docs` if not given (e.g. if the most recent in Supabase is Q3 2026, the upcoming is Q4 2026).
- **Audience hint** — optional; default `analyst-editorial`.

If the filer is missing, ask:
- "Which filer's earnings should I preview?"

### Step 2: Verify and load

1. Look up the filer in `anta-supabase` — canonical name + `filer_operator_id`.
2. Pull the prior cycle's themes (`earnings_call_themes`), AI mentions, AI capex disclosures, vendor mentions, and any commissioned/published angles for this filer.
3. Find the anticipated call date — use IR page, web search, or `catalyst-calendar` output if already loaded. Set `source_doc.filing_date` to the anticipated date or null.

### Step 3: Invoke the skill

Use `skill: "earnings-preview"` to build `listen_for` items and `prior_cycle_threads`.

### Step 4: Deliver output

Provide three things:

1. **Markdown summary table** — for `listen_for`, sorted by `priority` desc:

   | # | Topic | Category | Priority | Why | Speaker to watch |
   |---|---|---|---|---|---|

   And a separate table for `prior_cycle_threads`:

   | # | Thread | Prior cycle | What to watch | Resolves if |
   |---|---|---|---|---|

2. **JSON object** matching the schema in the skill's `SKILL.md` — `source_doc` envelope + `listen_for` + `prior_cycle_threads` + `summary`.

3. **4–6 sentence editorial summary** for the commissioning editor. Lead with the most editorially loaded thread.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to pull the catalyst calendar entry for this date to cross-check?"
- "Want me to draft the post-call follow-on plan: which themes from this preview would trigger a `/tv-angle` if confirmed?"
- "Want me to check `/peer-set` for what the comparable peers said on the same threads in their last calls?"

## Quality Checklist

Before delivery, confirm:
- [ ] Filer matched to ANTA canonical name; `filer_kind` correctly set; `filer_operator_id` populated for operator filers
- [ ] Anticipated `filing_date` captured or explicitly null
- [ ] 3–7 `listen_for` items, each with category + priority + why + specific questions
- [ ] Every `prior_cycle_thread` has a verbatim Supabase-sourced prior quote + falsifiable `resolves_if`
- [ ] Priority distribution honest — not everything is `high`
- [ ] If prior-cycle data is thin, said so explicitly in the summary
- [ ] No equity-research artefacts (consensus, whisper numbers, scenarios with stock-price reactions)
