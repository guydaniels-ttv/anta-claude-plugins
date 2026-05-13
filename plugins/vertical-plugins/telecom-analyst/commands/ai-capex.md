---
description: Extract AI-related capex and capex-adjacent disclosures from an operator or vendor filing or transcript — quantified spending claims with status, scope, and trajectory
argument-hint: "[path or URL to source doc] [filer name, optional] [cycle, optional]"
---

# AI Capex Tracker Command

Scan a single primary-source document for every quantified AI-spending disclosure. Classify by status (spent / committed / announced / aspirational), AI scope, and capex/opex. Compare to prior cycles.

## Workflow

### Step 1: Parse the inputs

- **Source doc** — path or URL to a filing PDF/HTML, transcript, investor deck, or press release
- **Filer name** — optional; infer from the doc if not given
- **Cycle** — optional; infer from the doc if not given

If the source doc is missing, ask:
- "Which document should I scan? Paste a path or URL."

### Step 2: Verify and load

1. Fetch the doc via `filings-store` MCP if a URL.
2. Verify the doc is the latest cycle.
3. Look up the filer in `anta-supabase` for canonical name.
4. Query `anta-supabase` for prior `ai_capex` disclosures for this filer (for the comparison step).
5. Pull the ANTA vendor universe for cross-checking named recipients.

### Step 3: Invoke the skill

Use `skill: "ai-capex-tracker"` to do the extraction and classification.

### Step 4: Deliver output

Provide three things:

1. **Markdown summary table** sorted by `disclosure_status` priority (spent > committed > announced > aspirational) then by `amount_normalized` desc:

   | # | Amount | Currency | Horizon | Status | Scope | Capex/Opex | % of total | Vendors | Confidence |
   |---|---|---|---|---|---|---|---|---|---|

2. **JSON object** matching the schema in the skill's `SKILL.md`, ready for Supabase ingest.

3. **3–6 sentence narrative** for a TelecomTV editor. Lead with the most newsworthy disclosure (usually the largest `spent` or `committed` figure, or a material revision). State explicitly if zero quantified disclosures were found — that is itself news.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to ingest this into the ANTA Supabase `ai_capex` table?"
- "Want me to also run `/ai-mentions` for the AI-content lens?"
- "Want me to run `/filing-diff --focus capex` against the prior cycle to find capex disclosures that have gone quiet?"
- "Want me to build a multi-cycle AI capex trajectory chart for this filer from Supabase history?"

## Quality Checklist

Before delivery, confirm:
- [ ] Source doc verified as latest cycle
- [ ] Filer matched to ANTA canonical name
- [ ] `total_capex_disclosed` populated only if explicitly in the doc — no external pulls
- [ ] Every AI-keyword + number proximity scanned, including footnotes and Q&A
- [ ] `amount_normalized` arithmetic correct for m/bn/cr/lakh suffixes; same-currency assumption checked
- [ ] `disclosure_status` honest — vague targets are `aspirational`, not `announced`
- [ ] `capex_or_opex` defaults to `unclear` when ambiguous; not silently coerced
- [ ] `vendors_involved` cross-checked against ANTA vendor universe
- [ ] `ratio_to_total_capex` only computed when currencies and horizons match
- [ ] Prior-cycle comparison done where Supabase has the data
- [ ] If zero disclosures found, that is stated explicitly in the summary
- [ ] Output JSON validates against the schema
