---
description: Extract every named third-party vendor, partner, hyperscaler, NEP, or AI infra supplier from an operator or vendor filing, transcript, or investor doc, and cross-check against the ANTA vendor universe
argument-hint: "[path or URL to source doc] [filer name, optional] [cycle, optional]"
---

# Vendor Mentions Command

Scan a single primary-source document and produce a structured table of every named third party — vendors, partners, suppliers, hyperscalers — classified by vendor type, relationship, and domain.

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
3. Look up the filer in `anta-supabase` — confirm canonical name and determine `filer_side` (operator or vendor).
4. Pull the ANTA vendor universe (canonical names + aliases + types) from `anta-supabase` to use as the seed list.

### Step 3: Invoke the skill

Use `skill: "vendor-mentions"` to do the extraction and classification.

### Step 4: Deliver output

Provide three things:

1. **Markdown summary table** sorted by `materiality` then `domain`:

   | # | Vendor | Type | Relationship | Domain | Materiality | AI? | Quote (truncated) |
   |---|---|---|---|---|---|---|---|

2. **JSON object** matching the schema in the skill's `SKILL.md`, ready for Supabase ingest.

3. **3–5 sentence narrative** for a TelecomTV editor — lead with the most consequential mention. Flag any `universe_candidates` (names not yet in ANTA's universe) explicitly.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to ingest this into the ANTA Supabase `vendor_mentions` table?"
- "Want me to also run `/ai-mentions` for the AI-content lens?"
- "Want me to run `/filing-diff --focus vendors` against the prior cycle to find vendors that have gone quiet?"
- "Want me to draft Supabase rows to **add** the universe candidates to the vendor universe?" (Pause for human review before any write.)

## Quality Checklist

Before delivery, confirm:
- [ ] Filer matched to ANTA canonical name; `filer_side` correctly set
- [ ] ANTA vendor universe pulled and used as the seed list
- [ ] Every named third party captured — including ones in footnotes, lists, and Q&A
- [ ] Unmatched names appear in `universe_candidates`, not silently dropped
- [ ] `ai_related` flag set per the rule in `SKILL.md`
- [ ] `materiality` distribution is honest (not everything is `named-strategic`)
- [ ] No anonymised references invented as named entities
- [ ] Output JSON validates against the schema
