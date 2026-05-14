---
description: Compare two cycles of an operator or vendor's filings/transcripts and surface what's new, removed, reworded, or materially changed
argument-hint: "[current doc path or URL] [prior doc path or URL, optional] [operator or vendor name, optional] [--focus all|ai|risks|guidance|capex|strategy|vendors]"
---

# Filing Diff Command

Run a semantic cycle-over-cycle comparison and produce a ranked change list for TelecomTV editorial triage.

## Workflow

### Step 1: Parse the inputs

- **Current doc** — required; path or URL to the new cycle's source doc
- **Prior doc** — optional; if omitted, attempt to find it via `anta-supabase` and `filings-store`
- **Operator/vendor name** — optional; derive from the doc if not given
- **`--focus`** — optional; one of `all` (default), `ai`, `risks`, `guidance`, `capex`, `strategy`, `vendors`

If the current doc is missing, ask:
- "Which document is the new cycle?"

If the prior doc can't be auto-located and isn't supplied, ask:
- "I can't find the prior-cycle doc. Paste a path or URL, or tell me to skip and treat everything as `new`."

### Step 2: Verify and load

1. Verify the current doc is the latest cycle.
2. Verify both docs are the same type (transcript vs transcript, annual vs annual). If not, flag and ask whether to proceed with the caveat.
3. Look up the operator/vendor in `anta-supabase` for canonical name and any prior structured extractions.

### Step 3: Invoke the skill

Use `skill: "filing-diff"` to run the comparison. For `--focus ai`, the skill will internally call `ai-mentions-extractor` on each doc (or reuse prior `ai_mentions` rows from Supabase if available).

### Step 4: Deliver output

Provide three things:

1. **Markdown summary table** sorted by editorial value:

   | # | Status | Topic | Section | Delta summary | Value |
   |---|---|---|---|---|---|

2. **JSON object** matching the schema in the skill's `SKILL.md`, ready for archive or downstream consumption.

3. **3–6 sentence editorial narrative** for a TelecomTV editor — lead with the single most consequential change. State explicitly if any `removed` changes are present (these are easy to miss).

### Step 5: Offer next steps

After delivering, offer:
- "Want me to draft a `/tv-angle` based on the high-value changes?"
- "Want me to extract the full `ai-mentions` for the current cycle?"
- "Want me to file this diff into the ANTA Supabase?"

## Quality Checklist

Before delivery, confirm:
- [ ] Both docs verified as same type and correct cycles
- [ ] Operator/vendor matched to ANTA canonical name
- [ ] `removed` changes are present in the output if any exist (don't silently drop)
- [ ] No `unchanged` rows are padding the report
- [ ] Each `delta_summary` says **what** changed, not just that something changed
- [ ] `editorial_value` distribution is honest (not everything is `high`)
- [ ] Output JSON validates against the schema
