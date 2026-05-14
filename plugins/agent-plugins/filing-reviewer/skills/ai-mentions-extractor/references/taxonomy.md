# AI Mentions Taxonomy — Worked Examples

Reference cases for classifying each mention. Read this when a quote is ambiguous.

## Strategy

Forward-looking. Not in production yet. The verb tense is the giveaway: *will, plans to, intends to, by 2027, our roadmap, our ambition*.

| Quote | Why |
|---|---|
| "We will deploy GenAI agents across all customer-care channels by the end of FY26." | Future tense, no current deployment claimed. |
| "Our strategy is to embed AI at every layer of the network." | Aspiration, no specifics on what is live. |
| "We plan to invest €500m in AI infrastructure over the next three years." | Forward capex commitment — strategy, not contribution. |

## Deployment

In production now. Specific system, specific function. Verb tense: *is live, has been rolled out, currently handles, runs across, serves*.

| Quote | Why |
|---|---|
| "Our self-care GenAI agent handles 60% of inbound tier-1 queries in Pakistan." | Live, specific function, specific market, quantified. |
| "We use machine-learning models to predict cell-site failures across our European footprint." | Present tense, named function. |
| "Our €5m pilot of AI-driven field-ops scheduling reduced truck rolls 18% in three Spanish regions." | A pilot is still deployment — flag `subcategory: pilot` in `notes`. Not contribution, because no claim of aggregate company impact. |

## Contribution

Claimed business impact at company-level. Usually appears in CFO/CEO prepared remarks or MD&A. Looking for: *contributed X to revenue/EBITDA, reduced opex by Y, drove Z pp of growth, saved €N*.

| Quote | Why |
|---|---|
| "AI-driven personalisation contributed 3pp to ARPU growth this quarter." | Aggregate impact claim with a number. |
| "Automation initiatives, including our ML cost-to-serve programme, reduced operating costs by 4% YoY." | Company-level cost claim. |
| "We expect AI-related revenue to reach €100m in FY27." | Forward claim → **strategy** with a number, not contribution. Contribution must be realised. |

## Hype

Marketing language with no specific system or outcome. The test: could a competitor say the exact same sentence with the exact same truth value? If yes → hype.

| Quote | Why |
|---|---|
| "We are an AI-first telco." | Slogan, not a claim. |
| "Leveraging the transformative power of generative AI to reimagine the customer experience." | No system, no outcome, no quantification. |
| "AI is core to our DNA." | Marketing boilerplate. |

**Recycled hype:** if the same sentence appeared in the prior cycle's filing/transcript verbatim or near-verbatim, still classify as hype but flag `notes: recycled from <prior cycle>`. This is editorially interesting.

## Risk

AI named as a risk factor. Usually in the risk-factors section of a 10-K / 20-F / annual report. Includes regulatory risk, competitive risk, technological risk, ethical/reputational risk.

| Quote | Why |
|---|---|
| "Failure to keep pace with developments in AI could materially impair our competitive position." | Competitive/tech risk. |
| "Emerging AI regulation in the EU could increase our compliance costs and limit our deployment options." | Regulatory risk. |
| "The use of AI in our customer-care operations exposes us to potential reputational and discrimination risks." | Ethical / reputational risk. |

## Hard edge cases

### Pilot vs. deployment vs. contribution

A quantified pilot is **deployment** (`subcategory: pilot`), not contribution. Contribution requires a claim of aggregate company-level financial impact.

### Strategy with numbers

"We plan to save €200m through AI by 2027" — still **strategy** (it's a forward target), not contribution. The number is the target, not the realised impact.

### Vendor name without a use case

"Our partnership with Microsoft Azure OpenAI continues to develop" — capture this, but classify as **hype** unless the surrounding context names a system or outcome. Populate `vendors_named: ["Microsoft Azure OpenAI"]`.

### Mentions in Q&A

Analyst Q&A is where hype gets pressed for specifics. A management answer that names a system and outcome under questioning is **deployment** or **contribution**, even if the prepared remarks gave the same topic as hype. Capture both, flag in `notes`.

### Negative or hedged mentions

"We have not yet seen material revenue contribution from generative AI" — this is editorially valuable. Capture as **contribution** with `confidence: high` and `quantitative: false`, and note the negative direction in `notes`. (The category is what the speaker is *commenting on*, not whether the comment is positive.)

### Multi-sentence mentions

If a single AI topic spans 2–3 sentences (e.g. CEO describing one deployment in detail), capture as **one mention** with the full quote (truncating to 500 chars if needed). Don't fragment.
