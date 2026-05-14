---
name: vendor-mentions
description: Extract every named third-party (vendor, partner, hyperscaler, NEP, AI infra supplier, system integrator, tower co, BSS/OSS provider) from an operator or vendor financial filing or earnings call transcript. Classify each by vendor type, relationship type (supplier / customer / strategic partner / acquired / divested / ecosystem), and domain (RAN / Core / AI infra / Customer ops / Cloud / Security / BSS / OSS / Tower / IT / Devices). Cross-check named entities against the ANTA vendor universe in Supabase and flag new candidates. Use when the user asks to extract / list / map / scan vendor mentions, partners, partnerships, suppliers, hyperscalers, AI vendors, vendor relationships, named third parties, or material agreements in a filing, transcript, annual report, 10-K, 20-F, quarterly, earnings call, or investor day.
---

# Vendor Mentions Extractor

Extract and classify **every** named third-party from a single telco operator or vendor source document. Designed to map the ecosystem around each filer — who they buy from, who they sell to, who they co-develop with — and to keep the ANTA vendor universe current.

This skill is the relationship lens. `ai-mentions-extractor` is the content lens. Mentions that are both AI-related **and** name a vendor will appear in both skills' outputs by design — they're foreign-keyed to the same source doc.

## When to Use

Use when the user requests:
- "Extract vendor mentions from [operator]'s Q3 transcript"
- "Who is [vendor] partnering with in this annual report?"
- "Map the vendor ecosystem from this 20-F"
- "List every hyperscaler / NEP / AI infra supplier named in this filing"
- "Find new vendor relationships in [operator]'s investor day"
- "What vendors has [operator] stopped naming this cycle?" → still use this skill, then call `filing-diff --focus vendors`

**Do NOT use if:**
- The user wants AI-content classification → use `ai-mentions-extractor`
- The user wants only AI-specific vendor mentions → use this skill, then filter on `domain` matching `AI infra` or `AI model`
- The user wants to add a new vendor to the universe → flag it in this skill's output as a candidate, but the actual add is a Supabase write done elsewhere

## Inputs

| Input | Required | Notes |
|---|---|---|
| Source document | Yes | Filing PDF/HTML, transcript, press release, or investor deck. URL → `WebFetch`; local path → `Read`. |
| Filer (operator or vendor) | If not derivable | The company whose filing this is. Cross-check against ANTA universe. |
| Reporting period | If not derivable | E.g. Q3 2026, FY 2025. Goes into `source_doc.reporting_period`. |
| Doc type | If not derivable | Goes into `source_doc.doc_type` — see envelope reference. Required for UPSERT. |
| ANTA scoring cycle | Optional | If the run is tied to an ANTA scoring cycle, pass the `scoring_cycle_id` UUID. |
| Filer side | Derived from `filer_kind` | `operator` (the filer is buying — named third parties are suppliers/partners) or `vendor` (the filer is selling — named operators are customers). Determines default relationship-type mapping. |

## Output

**Primary deliverable**: a JSON object with the shared `source_doc` envelope + a `mentions` array + a `universe_candidates` array + a markdown summary table + a 3–5 sentence editorial narrative. The envelope shape is shared with every other extraction skill — see [../../references/source-doc-envelope.md](../../references/source-doc-envelope.md) for the full convention.

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
    "reporting_currency": null,
    "extracted_by": "vendor-mentions v1"
  },
  "mentions": [
    {
      "id": 1,
      "vendor_name_raw": "Microsoft Azure OpenAI",
      "vendor_canonical": "Microsoft",
      "vendor_type": "hyperscaler",
      "in_anta_universe": true,
      "relationship_type": "supplier",
      "domain": "AI infra",
      "page_or_timestamp": "p. 14 / 37:22",
      "speaker": "CEO Kaan Terzioğlu",
      "quote": "We've extended our partnership with Microsoft Azure OpenAI for our customer-care GenAI agents across all markets.",
      "context_tags": ["genai", "customer-care", "multi-market"],
      "materiality": "named-strategic",
      "confidence": "high",
      "ai_related": true,
      "notes": "Same partner as Q2 2026, scope expanded from Pakistan-only to all markets."
    }
  ],
  "universe_candidates": [
    {
      "mention_id": 7,
      "vendor_name_raw": "Algotive",
      "context_quote": "We have partnered with Algotive for our RAN energy-optimisation pilot in Bangladesh.",
      "suggested_vendor_type": "AI infra / silicon",
      "suggested_domain": "RAN"
    }
  ],
  "summary": "Nine vendor mentions, dominated by AI-related hyperscaler and infra suppliers. Headline: Microsoft Azure OpenAI partnership scope expanded from one market to all. One name new to the ANTA universe (Algotive — RAN AI pilot)."
}
```

The `filer_side` denormalised column on each `vendor_mentions` DB row is set by the loader from the envelope's `filer_kind` — the skill does not need to emit it per-mention.

`mention_id` on a `universe_candidates` entry is the `id` of the mention in this same output's `mentions` array that produced the candidate. The loader translates this to a `vendor_mention_id` FK at insert time. Set to `null` if the candidate isn't tied to a specific mention.

## Vendor Taxonomy

Eight vendor types. Pick the strongest fit. If a vendor straddles two (e.g. Microsoft as both hyperscaler and AI model provider via OpenAI), use the **role they play in this specific mention**.

| Vendor type | Examples | When this type |
|---|---|---|
| **hyperscaler** | Microsoft Azure, AWS, Google Cloud, Oracle Cloud, Alibaba Cloud | Cloud infra at hyperscale; named for cloud, AI infra, or platform services. |
| **NEP** | Ericsson, Nokia, Huawei, Samsung Networks, ZTE, Mavenir, Parallel Wireless | Network-equipment provider — RAN, Core, Transport. |
| **AI infra / silicon** | NVIDIA, AMD, Broadcom, Intel (data-center), Cerebras, Groq | Compute substrate for AI workloads. |
| **AI model provider** | OpenAI, Anthropic, Cohere, Mistral, Google DeepMind, AI21 | Foundation-model providers. |
| **BSS/OSS** | Amdocs, Netcracker, CSG, Salesforce, ServiceNow, Pegasystems, Oracle (apps) | Business and operations support systems. |
| **Tower / passive infra** | Cellnex, American Tower, Crown Castle, IHS, Vantage Towers, Indus Towers | Tower-co or passive-infra. |
| **Security** | Palo Alto, Fortinet, Cisco Security, CrowdStrike, Zscaler | Network and endpoint security. |
| **Integrator / consultant** | Accenture, Deloitte, IBM Consulting, Capgemini, Tech Mahindra, Cognizant | Systems integration, transformation services. |
| **Device / CPE** | Apple, Samsung Mobile, Huawei (devices), Xiaomi, Nokia (phones) | Handsets, modems, CPE. |
| **Other** | Anything that doesn't fit | Use sparingly; if you find yourself using `other` a lot, propose a new type in the universe candidates list. |

See [references/vendor-taxonomy.md](references/vendor-taxonomy.md) for edge cases — companies that span types (Samsung, Huawei, Oracle), and the rule for which type to pick per-mention.

## Relationship Types

Six relationship types. Default depends on the **filer side**:

- For an **operator** filer (e.g. Vodafone, Orange, VEON), named third parties are typically `supplier` or `strategic-partner`.
- For a **vendor** filer (e.g. Ericsson, Nokia, Amdocs), named operators are typically `customer`.

| Relationship | Definition |
|---|---|
| **supplier** | Filer buys from the named party. Default for operator filings. |
| **customer** | Named party buys from the filer. Default for vendor filings. |
| **strategic-partner** | Joint development, JV, co-go-to-market. Bilateral. |
| **acquired** | Filer has acquired (or is acquiring) the named party. |
| **divested** | Filer has divested (or is divesting) the named party. |
| **ecosystem** | Loose, non-bilateral mention. E.g. "members of the broader Open RAN ecosystem including X, Y, Z." |

## Domains

The functional area where the vendor sits. One per mention.

`RAN` · `Core` · `Transport / IP` · `Customer ops` · `Billing / BSS` · `OSS` · `AI infra` · `AI model` · `Cloud` · `Security` · `Devices / CPE` · `Tower / Infra` · `IT` · `Field ops` · `M&A / corporate` · `Other`

## Materiality

Three levels.

| Level | Definition |
|---|---|
| **named-strategic** | Explicit strategic partner, multi-year, material to the filer's strategy. Often quantified or scoped (e.g. "across all markets," "primary supplier"). |
| **named-operational** | Contract or agreement named with operational scope. Specific, but not framed as strategy-defining. |
| **passing-mention** | Vendor named in passing — list of ecosystem members, generic reference, footnote. |

## Trigger Patterns

When scanning, look for these phrasal patterns:

- "partnership with X" / "agreement with X" / "deal with X"
- "X has been selected" / "we have selected X"
- "X provides our…" / "we use X for…" / "we deploy X to…"
- "in collaboration with X" / "in partnership with X"
- "powered by X" / "built on X"
- "our suppliers" + bullet list
- "material agreements" section in filings — lists explicit
- "ecosystem partners include X, Y, Z"
- M&A: "we acquired X" / "we sold X to" / "merger with X"

Also pass the **ANTA vendor universe** (from `anta-supabase`) as a seed list — scan the doc for any known canonical name or alias.

## Workflow

### Step 1: Identify and verify

1. Confirm doc is the latest cycle (filing date captured).
2. Identify the filer; determine `filer_kind` (operator or vendor) by looking it up in the ANTA universe. For operator filers, capture `filer_operator_id`.
3. Determine `doc_type` from the 12-value enum (annual_report / 10-K / 20-F / quarterly_report / earnings_call_transcript / investor_day / press_release / investor_deck / analyst_day / cmd / regulatory_filing / other). Required for UPSERT.
4. Capture `reporting_period` and `scoring_cycle_id` (if the run is bound to an ANTA cycle). Populate the full `source_doc` envelope before extracting mentions.

### Step 2: Locate vendor sections

Expect vendor content in:

**Filings:**
- "Material agreements" or "Material contracts" section (explicit list)
- MD&A — partnerships and capex narrative
- "Customers" section (for vendor filers)
- Risk factors — supply concentration risks
- Notes on commitments

**Transcripts:**
- CEO prepared remarks — strategic partnerships
- CFO prepared remarks — supply/cost commentary
- Analyst Q&A — pressure-tested specifics

**Press releases / investor decks:**
- Often dedicated to a single named partnership — the whole doc is one mention.

### Step 3: Pull the ANTA vendor universe

Query `anta-supabase` for the canonical vendor list with aliases and types. Use this as the seed list to scan against. Where a named entity matches, set `vendor_canonical` and `vendor_type` from the universe row.

### Step 4: Extract each mention

For each match (universe-seeded or pattern-matched):
1. Capture the verbatim quote (≤500 chars)
2. Capture page or timestamp + speaker (transcripts)
3. Set `vendor_canonical` from the universe match (or `null` if unmatched)
4. Set `in_anta_universe` accordingly
5. Classify `vendor_type`, `relationship_type`, `domain`
6. Set `materiality`
7. Capture `context_tags` — lowercase short tags (e.g. `genai`, `customer-care`, `pilot`, `multi-market`, `multi-year`)
8. Set `ai_related: true` if domain ∈ {`AI infra`, `AI model`} or vendor_type ∈ {`AI infra / silicon`, `AI model provider`} or context references AI/GenAI/ML

### Step 5: Universe candidates

For each named entity **not** matched to the ANTA universe, add to `universe_candidates`:
- `mention_id` — the `id` of the mention in this output's `mentions` array that produced the candidate (or `null` if not tied to a specific mention)
- `vendor_name_raw` — the raw name as it appeared
- `context_quote` — the context quote that anchors it (≤500 chars)
- `suggested_vendor_type` — one of the vendor_type enum values (informed guess)
- `suggested_domain` — one of the domain enum values (informed guess)

The loader uses `mention_id` to populate the `vendor_mention_id` FK on `vendor_universe_candidates` so a reviewer can jump straight from candidate to its originating mention. Candidates flow downstream to a human reviewer who decides whether to promote into the (future) ANTA vendor universe.

### Step 6: Summary

Emit a 3–5 sentence narrative for an editor. Lead with the single most consequential mention. Note:
- Total count and the dominant vendor types
- Any new universe candidates (especially if AI-related)
- Any prior-cycle vendor that **didn't** appear this cycle — but defer the actual removed-list to `filing-diff --focus vendors`. Just flag the suspicion.

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
    "reporting_currency": null,
    "extracted_by": "vendor-mentions v1"
  },
  "mentions": [
    {
      "id": "integer",
      "vendor_name_raw": "string — as it appeared in the doc",
      "vendor_canonical": "string or null — from ANTA universe",
      "vendor_type": "hyperscaler | NEP | AI infra / silicon | AI model provider | BSS/OSS | Tower / passive infra | Security | Integrator / consultant | Device / CPE | Other",
      "in_anta_universe": "boolean",
      "relationship_type": "supplier | customer | strategic-partner | acquired | divested | ecosystem",
      "domain": "RAN | Core | Transport / IP | Customer ops | Billing / BSS | OSS | AI infra | AI model | Cloud | Security | Devices / CPE | Tower / Infra | IT | Field ops | M&A / corporate | Other",
      "page_or_timestamp": "string or null",
      "speaker": "string or null",
      "quote": "string ≤500 chars, verbatim",
      "context_tags": ["string", ...],
      "materiality": "named-strategic | named-operational | passing-mention",
      "confidence": "high | medium | low",
      "ai_related": "boolean",
      "notes": "string — one sentence"
    }
  ],
  "universe_candidates": [
    {
      "mention_id": "integer or null — id of the originating mention in this output's mentions array",
      "vendor_name_raw": "string",
      "context_quote": "string ≤500 chars — quote anchoring the mention",
      "suggested_vendor_type": "string — one of the vendor_type values",
      "suggested_domain": "string — one of the domain values"
    }
  ],
  "summary": "string — 3 to 5 sentences"
}
```

## Quality Checklist

Before delivery:
- [ ] `source_doc` envelope is complete and valid (filer_kind set, filer_name canonical, doc_type set, reporting_period set)
- [ ] `filer_operator_id` populated only when `filer_kind == 'operator'` AND a match exists; else null
- [ ] Source doc verified as latest cycle (filing_date captured)
- [ ] ANTA vendor universe pulled and used as seed list
- [ ] Every named third party captured — including ones in lists, footnotes, and Q&A
- [ ] `vendor_canonical` set from the universe where a match was found
- [ ] `in_anta_universe` is correct for every row
- [ ] Unmatched names appear in `universe_candidates`, not silently dropped
- [ ] Each `universe_candidates` entry uses `context_quote` and `suggested_vendor_type` (not the older `context` / `suggested_type` field names)
- [ ] `ai_related` flag set correctly (use the rule above, don't guess)
- [ ] `materiality` is honestly assessed — not everything is `named-strategic`
- [ ] `filer_kind` drives sensible defaults for `relationship_type`
- [ ] Output JSON validates against the schema

## Resources

### references/vendor-taxonomy.md
Worked examples for the harder vendor-type and relationship-type calls — companies that span types (Samsung, Oracle, Huawei), and how to handle JVs / minority investments / reseller relationships.

### ../../references/source-doc-envelope.md
The shared `source_doc` envelope convention used across all telecom-analyst extraction skills.

## Dependencies

**Required:**
- `WebFetch` (URLs) / `Read` (local paths) — to fetch source documents
- `anta-supabase` MCP — to pull the canonical vendor universe (names, aliases, types) and the operator universe (for filer-side resolution)

**Optional:**
- `ai-mentions-extractor` skill — when the user wants both lenses on the same doc, run both
- `filing-diff` skill — to surface vendors that have appeared / disappeared cycle-over-cycle
