# Vendor Taxonomy — Hard Cases

Reference for the harder vendor-type and relationship-type calls. Read this when the SKILL.md taxonomy table doesn't obviously fit.

## Companies that span types

Pick the type matching the **role they play in this specific mention**. Same company can be different types in different mentions.

### Microsoft

- Mention names Azure cloud / Office 365 / Teams / Dynamics → **hyperscaler**
- Mention names Azure OpenAI Service / Copilot / GPT-via-Azure → **AI model provider** (because the model is what's being bought) OR **hyperscaler** if the framing is infra-led. When in doubt → **hyperscaler** (Microsoft's primary identity for telcos).
- Mention names the company generically ("our Microsoft partnership") → **hyperscaler**

### Google

- Mention names Google Cloud / GCP / Vertex AI infra → **hyperscaler**
- Mention names Gemini / DeepMind / Google AI specifically → **AI model provider**
- YouTube / Android partnerships → **Other** (ecosystem)

### NVIDIA

- Always **AI infra / silicon** unless the mention is specifically about networking gear (Mellanox / BlueField for fronthaul) → could be NEP, but rare. Default AI infra.

### Samsung

- Samsung Networks (RAN) → **NEP**
- Samsung Mobile (handsets) → **Device / CPE**
- Samsung Cloud → unlikely in telco filings; **Other** if it appears

### Huawei

- Default → **NEP** (the most common context for telco filings)
- Huawei Cloud → **hyperscaler**
- Huawei devices → **Device / CPE**

### Oracle

- Oracle Cloud Infrastructure → **hyperscaler**
- Oracle apps (Siebel, EBS, ERP, comms apps) → **BSS/OSS**
- Oracle databases / generic "Oracle" → **Other** unless context disambiguates

### IBM

- IBM Cloud → **hyperscaler**
- IBM Consulting / Kyndryl-related → **Integrator / consultant**
- watsonx / IBM AI offerings → **AI model provider**

### Ericsson / Nokia

- Default → **NEP**
- Ericsson Vonage (APIs / CPaaS) → **Other** — flag as a sub-domain note
- Nokia Bell Labs research mentions → **Other**

## Joint ventures and minority investments

- A JV announcement names the JV partner → `relationship_type: strategic-partner` (the JV itself is *not* the vendor type — pick a type for the partner)
- Minority investment ("we have taken a 15% stake in X") → `relationship_type: strategic-partner`, note the stake in `notes`
- Spin-off / carve-out where the filer retains a stake → `relationship_type: divested`, note retained stake in `notes`

## Resellers, white-label, and bundled offerings

If the filer **resells** another vendor's product (e.g. a telco reselling AWS to enterprise customers), the third party is still a `supplier` from the filer's perspective. The reseller relationship doesn't change the type — but flag it in `context_tags` (e.g. `reseller`, `white-label`).

## M&A mentions

When the filer references M&A activity:

| Scenario | relationship_type | Notes |
|---|---|---|
| Filer acquired (closed) the named party | `acquired` | Set `domain: M&A / corporate` |
| Filer is acquiring (pending) the named party | `acquired` | Flag pending status in `notes` |
| Filer divested (closed) the named entity | `divested` | Set `domain: M&A / corporate` |
| Filer is being acquired by named party | This is a parent-mention, not a vendor mention — skip unless the named acquirer plays a vendor role elsewhere in the doc |

## Customer mentions in vendor filings

When the filer is a vendor (Ericsson, Nokia, Amdocs, etc.), they will name operator customers. Each named operator is a vendor-mention of `relationship_type: customer`.

- If the named operator is in the ANTA operator universe → set `vendor_canonical` to that canonical name. The "vendor" field is being used loosely here — the named party is a customer, but we still want it cross-referenced.
- `vendor_type` for an operator-as-customer doesn't really apply — set it to `Other` and use `context_tags: [operator-customer]` to flag.

The Supabase migration will need to handle this — operators and vendors share a "named party" namespace in this skill's output. Worth noting for the table design.

## Ecosystem mentions

A list like "ecosystem partners including X, Y, Z, and A" is a passing-mention. Each named party gets its own row with `materiality: passing-mention` and `relationship_type: ecosystem`. Don't lump them into one row.

## Generic supplier references

"Our suppliers" or "leading hyperscalers" without naming them → **don't create a row.** This skill is for *named* third parties only. If the user wants generic supplier mentions, that's a different skill.

## Anonymised mentions

"A major North American hyperscaler" — don't create a row. If the doc names them elsewhere, that named mention is the row. If not, note in the summary that anonymised references exist but don't fabricate names.

## Confidence calibration

- **high** — clear name, clear role, clear domain. Universe-match strong.
- **medium** — name is clear, but the role or domain requires interpretation. Or the universe-match is by alias and worth a human eye.
- **low** — partial name, ambiguous role, multiple plausible interpretations. Always pair with detailed `notes` so a human reviewer can adjudicate.
