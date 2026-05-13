---
description: Extract and classify every AI / GenAI / ML / agent mention from an operator or vendor filing, transcript, or investor doc
argument-hint: "[path or URL to source doc] [operator or vendor name, optional] [cycle, e.g. Q3 2026, optional]"
---

# AI Mentions Extractor Command

Scan a single primary-source document and produce a structured table of every AI-related mention, classified for ANTA's tracking.

## Workflow

### Step 1: Parse the inputs

Parse the user's invocation for:
- **Source doc** — path or URL to a filing PDF/HTML, earnings call transcript, investor day deck, or press release
- **Operator or vendor name** — optional; infer from the doc if not given
- **Cycle** — optional; infer from the doc if not given (e.g. "Q3 2026", "FY 2025")

If the source doc is missing, ask:
- "Which document should I scan? Paste a path or URL."

### Step 2: Verify and load

1. If the source is a URL, fetch via the `filings-store` MCP or web search.
2. Verify the doc is the latest cycle, not stale training data — confirm the filing date.
3. Look up the operator/vendor in `anta-supabase` to get the canonical name and prior-cycle mentions if available.

### Step 3: Invoke the skill

Use `skill: "ai-mentions-extractor"` to do the actual extraction and classification.

### Step 4: Deliver output

Provide three things:

1. **Markdown summary table** for editorial review:

   | # | Category | Subcategory | Quote (truncated) | Vendors | Quant? | Confidence |
   |---|---|---|---|---|---|---|

2. **JSON array** matching the schema in the skill's `SKILL.md`, ready for Supabase ingest.

3. **3–5 sentence narrative** for a TelecomTV editor: total mention count, category mix, the single most interesting mention this cycle, and any meaningful cycle-over-cycle delta.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to ingest this into the ANTA Supabase `ai_mentions` table?"
- "Want me to run `/filing-diff` against the prior cycle?"
- "Want me to draft a `/tv-angle` for the editorial team?"

## Quality Checklist

Before delivery, confirm:
- [ ] Source doc verified as latest cycle
- [ ] Operator/vendor matched to ANTA canonical name
- [ ] All trigger-keyword classes were searched (not just "AI")
- [ ] Every mention has verbatim quote + location reference
- [ ] No `hype` mention is silently dropped — boilerplate matters
- [ ] `vendors_named` populated and cross-checked against ANTA universe
- [ ] Prior-cycle comparison done where data is available in Supabase
- [ ] Output JSON validates against the schema
