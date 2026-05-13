---
name: ai-mentions-extractor
description: Extract every AI / GenAI / ML / agent / automation / autonomous mention from an operator or vendor financial filing or earnings call transcript. Classify each mention as strategy, deployment, contribution, hype, or risk; capture vendors named; flag whether the claim is quantified. Use when the user asks to scan, extract, list, or analyse AI mentions / AI strategy / AI references / AI deployment / GenAI / generative AI in a filing, transcript, annual report, 10-K, 20-F, quarterly, earnings call, investor day, or press release. The primary output is a structured table that ANTA can ingest into Supabase.
---

# AI Mentions Extractor

Extract and classify **every** AI-related mention from a single telco operator or vendor source document. Optimised for ANTA's tracking of AI strategy, deployment, and revenue/cost contribution across the global telco universe.

This skill produces structured data — one row per mention — designed to be appended to ANTA's Supabase. It is not a narrative summary skill (that's the `morning-note` / `earnings-analysis` job downstream).

## When to Use

Use when the user requests:
- "Extract AI mentions from [operator]'s Q3 results"
- "What is [vendor] saying about AI in this annual report?"
- "Scan this transcript for AI strategy / deployment / contribution"
- "Pull every GenAI reference out of this 10-K"
- "AI mentions in [operator]'s investor day"

**Do NOT use if:**
- The user wants a narrative thesis update → use `thesis-tracker`
- The user wants AI capex specifically → use `ai-capex-tracker`
- The user wants vendor partner mapping specifically → use `vendor-mentions`
- The source isn't a single primary document (e.g. "tell me about AI in telecoms generally") → search the ANTA Supabase / TelecomTV archive first

## Inputs

| Input | Required | Notes |
|---|---|---|
| Source document | Yes | Path to a filing PDF/HTML, transcript, press release, or investor deck. If the user gives a URL, fetch via `filings-store` MCP or web. |
| Operator/vendor name | If not derivable from the doc | Cross-check against the ANTA universe in the `anta-supabase` MCP. |
| Cycle (e.g. Q3 2026, FY 2025) | If not derivable from the doc | Used for the `cycle` field in the output. |

## Output

**Primary deliverable**: a JSON array (one object per mention) + a markdown summary table for human review. Each mention object:

```json
{
  "operator_or_vendor": "VEON",
  "cycle": "Q3 2026",
  "source_doc": "VEON Q3 2026 earnings call transcript",
  "source_url": "https://...",
  "page_or_timestamp": "p. 14 / 37:22",
  "speaker": "CEO Kaan Terzioğlu",
  "quote": "We've rolled out our self-care GenAI agent in Pakistan, handling 60% of inbound tier-1 queries with a 35% deflection rate from human agents.",
  "category": "deployment",
  "subcategory": "customer ops / self-care",
  "vendors_named": ["Microsoft Azure OpenAI"],
  "quantitative": true,
  "confidence": "high",
  "notes": "Concrete production deployment with quantified KPIs; CEO-attributed, low ambiguity."
}
```

Also emit a one-paragraph **summary narrative** for the editorial team: mention count, dominant category, anything genuinely new vs. prior cycle (cross-check ANTA Supabase if the prior cycle is available).

## Classification Taxonomy

Five mutually exclusive categories. Pick the *strongest* fit; if it could plausibly be two, prefer the more concrete (deployment > strategy > hype).

| Category | What it is | Example |
|---|---|---|
| **strategy** | Forward-looking intent, vision, plans, targets. Not yet in production. | "We will deploy GenAI across all customer-service channels by end-FY26." |
| **deployment** | Concrete in-production use. Specific system, specific function, present-tense. | "Our network anomaly-detection model is live across 12 markets." |
| **contribution** | Claimed business impact — revenue, cost, productivity, churn, ARPU. Usually quantified. | "AI-driven personalisation contributed 3pp to ARPU growth this quarter." |
| **hype** | Vague, unsupported, marketing language. No specific system, no specific outcome. | "We are an AI-first telco, leveraging the power of intelligence at every layer." |
| **risk** | AI named as a risk — regulatory, competitive, technological, ethical. Usually in the risk-factors section. | "Failure to keep pace with AI developments could materially impair our competitive position." |

**Edge cases:**
- A quantified pilot ("AI saved €5m on field-ops trial in Spain") with no production rollout → **deployment** (subcategory "pilot"), not contribution. Contribution requires a claim of *aggregate company* impact.
- Strategy + numbers ("we plan to invest €500m in AI over three years") → still **strategy**, not contribution. Forward-looking commitment, not realised impact.
- Boilerplate hype recycled across multiple quarters → still **hype**, but flag in `notes` as "recycled from prior cycle."

See [references/taxonomy.md](references/taxonomy.md) for additional worked examples.

## Trigger Keywords

When scanning a document, search case-insensitively for:

- Core: `AI`, `A.I.`, `artificial intelligence`
- GenAI: `GenAI`, `generative AI`, `LLM`, `large language model`, `foundation model`
- ML: `machine learning`, `ML`, `deep learning`, `neural network`
- Agents/automation: `agent`, `agentic`, `autonomous`, `copilot`, `assistant`, `chatbot`
- Telco-specific: `AI-RAN`, `network AI`, `intent-based networking`, `self-healing`, `cognitive network`, `digital twin`
- Capabilities: `personalisation`, `predictive`, `anomaly detection`, `recommendation engine`, `forecasting model`
- Partners: `OpenAI`, `Anthropic`, `Microsoft Copilot`, `Azure OpenAI`, `Google Gemini`, `Vertex AI`, `AWS Bedrock`, `Cohere`, `Mistral`, `NVIDIA`

For each hit, expand to the **full sentence plus the sentence before and after** for context before classifying. A single sentence in isolation often misclassifies.

## Workflow

### Step 1: Identify and verify the source

1. Confirm the doc is the latest cycle (not a stale training-data version). Note the filing date.
2. Identify the operator/vendor. Look it up in `anta-supabase` to get the canonical name and any prior-cycle context.
3. Note the cycle (Q-Y or FY-Y).

### Step 2: Locate AI sections

Don't read end-to-end. Scan for the trigger keywords above. In filings, expect AI content in:

- MD&A / business overview
- Strategy or "our differentiation" section
- Risk factors (for `risk` category)
- Capex / investment commentary
- Notes on partnerships and material agreements

In transcripts, expect:
- CEO's prepared remarks (highest density of strategy + hype)
- CFO's prepared remarks (more contribution and capex)
- Analyst Q&A (often the only place hype gets pressed for specifics)

### Step 3: Extract each mention

For each hit:
1. Capture the verbatim quote (≤500 chars; trim mid-sentence with `…` if needed)
2. Capture page number (filings) or timestamp/speaker (transcripts)
3. Capture surrounding context to inform classification
4. Move to step 4

### Step 4: Classify

Apply the taxonomy. Set `confidence`:
- **high** — unambiguous, single category fits cleanly
- **medium** — fits one category but could plausibly be argued otherwise
- **low** — ambiguous, possibly multiple categories, or quote is partial

Capture any third-party names mentioned in `vendors_named` (e.g. "Microsoft Azure OpenAI", "NVIDIA", "Ericsson"). Cross-check against the ANTA vendor universe via `anta-supabase` and use the canonical name if matched.

### Step 5: Comparison and summary

If `anta-supabase` has the prior cycle for this operator/vendor:
- Note new mentions (vendors, subcategories, claims) vs. last cycle
- Note removed mentions (i.e. things they were boasting about a quarter ago that have gone quiet)
- Note recycled boilerplate (hype repeated verbatim)

Emit a 3–5 sentence summary covering: total mention count, category mix, the single most editorially interesting mention, and any cycle-over-cycle delta worth flagging to a TelecomTV editor.

## Output Schema

JSON array of objects, each conforming to:

```json
{
  "operator_or_vendor": "string (canonical name from ANTA universe)",
  "cycle": "string, e.g. 'Q3 2026' or 'FY 2025'",
  "source_doc": "string, human-readable",
  "source_url": "string, optional",
  "page_or_timestamp": "string, optional",
  "speaker": "string, optional (transcripts only)",
  "quote": "string, ≤500 chars, verbatim",
  "category": "strategy | deployment | contribution | hype | risk",
  "subcategory": "string, short free-form tag, e.g. 'customer ops', 'RAN', 'capex'",
  "vendors_named": ["string", ...],
  "quantitative": "boolean",
  "confidence": "high | medium | low",
  "notes": "string, one sentence"
}
```

Plus a markdown rendering of the same data as a table for human review, and the 3–5 sentence summary narrative.

## Quality Checklist

Before delivery:
- [ ] Source document is the latest cycle (date verified)
- [ ] Operator/vendor matched to ANTA canonical name
- [ ] Every trigger keyword class was searched (not just "AI")
- [ ] Every mention has verbatim quote + location reference
- [ ] No `hype` mention is missing — be honest about boilerplate
- [ ] `quantitative` flag is true wherever the quote contains numbers, percentages, or currency
- [ ] `vendors_named` is populated where applicable, cross-checked against ANTA universe
- [ ] Prior-cycle comparison done if data available in Supabase
- [ ] Output JSON is valid and schema-conformant

## Resources

### references/taxonomy.md
Worked examples for each category, including hard edge cases and "looks like X but is actually Y" patterns.

## Dependencies

**Required:**
- `filings-store` MCP — to fetch the source document
- `anta-supabase` MCP — to look up canonical operator/vendor names, vendor universe, and prior-cycle mentions

**Optional:**
- `telecomtv-archive` MCP — for editorial context on prior coverage
