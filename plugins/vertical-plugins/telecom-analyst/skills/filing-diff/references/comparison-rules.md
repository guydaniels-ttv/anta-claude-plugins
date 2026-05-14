# Comparison Rules — Worked Examples

Reference cases for classifying each change status. Read this when a pairing is ambiguous.

## new

Topic present this cycle, no semantic counterpart in prior cycle.

| Prior cycle | Current cycle | Status | Why |
|---|---|---|---|
| (no mention of AI in revenue contribution) | "AI-driven personalisation contributed 3pp to ARPU growth this quarter." | **new** | First time the contribution claim has been made. |
| (no mention of NVIDIA) | "We've signed a multi-year agreement with NVIDIA for our AI infrastructure." | **new** | New named partner. |
| (no risk factor on AI regulation) | "Emerging AI regulation in the EU could increase our compliance costs." | **new** | Risk factor added — editorially meaningful. |

## removed

Topic present in prior cycle, no semantic counterpart this cycle. **Often the most editorially interesting bucket.**

| Prior cycle | Current cycle | Status | Why |
|---|---|---|---|
| "We continue to expand 5G FWA across Ukraine despite ongoing challenges." | (no mention) | **removed** | Was a featured strategic priority; silence is meaningful. |
| "Our partnership with [Vendor X] is delivering strong results across our European network." | (no mention) | **removed** | Named partner has gone quiet — possible breakdown of relationship. |
| "We expect AI to be a meaningful revenue contributor in FY26." | (no mention of AI contribution) | **removed** | Forward claim made, then not reiterated when the period actually arrived. |

**Be honest about removals.** Don't rationalise them as "probably just not mentioned this quarter" — that's the editorially interesting fact.

## reworded

Same topic, same substance, materially different language. Capture both quotes; the substance is unchanged but the framing has shifted.

| Prior cycle | Current cycle | Status | Why |
|---|---|---|---|
| "We are deploying GenAI agents across our customer-care function in Pakistan." | "We have deployed GenAI agents across our customer-care function in Pakistan." | **reworded** | Tense shift only — same scope, same market, same system. |
| "Our AI strategy is to embed intelligence at every layer of the network." | "Intelligence is now embedded across every layer of our network." | **reworded** | Wording shifted from intent to claim, but no specifics added — substance is the same boilerplate. |
| "Generative AI represents one of the most significant opportunities of our generation." | "We see generative AI as a once-in-a-generation opportunity." | **reworded** | Recycled hype with cosmetic rewrite. Often low editorial value. |

**Reworded ≠ unchanged.** If the language has been deliberately rewritten, that's worth knowing even when the substance hasn't moved — it can signal an internal messaging change. Rate editorial value `low` for cosmetic rewords, `medium` if the framing has shifted meaningfully (e.g. intent → claim).

## materially-changed

Same topic, substantive change — number moved, scope expanded/narrowed, partner swapped, commitment hardened or softened.

| Prior cycle | Current cycle | Status | Why |
|---|---|---|---|
| "Our self-care GenAI agent handles 45% of inbound tier-1 queries in Pakistan." | "Our self-care GenAI agent handles 60% of inbound tier-1 queries in Pakistan." | **materially-changed** | KPI moved 15pp. |
| "We plan to invest €500m in AI infrastructure over the next three years." | "We plan to invest €750m in AI infrastructure over the next four years." | **materially-changed** | Commitment scaled up; also extended in time. |
| "Our infrastructure partnership with Microsoft Azure OpenAI continues to develop." | "We have shifted our primary AI infrastructure to NVIDIA and AWS Bedrock." | **materially-changed** | Partner swap — strong editorial signal. |
| "We expect capex to be in the range of €4.5–4.8bn for FY26." | "We expect capex to be in the range of €4.2–4.5bn for FY26." | **materially-changed** | Guidance lowered. |

## reworded vs materially-changed — the hard call

The most common judgement call. The test:

> *Has any number, scope, partner, market, system, function, timeline, or commitment level changed?*

- If **yes** to any → `materially-changed`
- If **no** to all and only the framing/tense/word choice has shifted → `reworded`

When in doubt → **`materially-changed`**. Under-classifying real change is worse than over-classifying.

## unchanged (omit from output)

Substantively identical, including verbatim or near-verbatim recurring boilerplate. **Omit from the output array.** Don't pad the report.

Common cases:
- Standard safe-harbour / forward-looking-statements disclaimer
- Long-standing risk factors copy-pasted across cycles
- CEO sign-off boilerplate ("we remain confident in our strategy…")
- Recycled hype (the same "AI-first telco" line in successive quarters)

If you classified a mention as recycled-hype in `ai-mentions-extractor`, it should appear here as `unchanged` and be omitted from the diff output — but the original recycled-hype flag in the mention record is still useful downstream.

## Editorial value — the call

Three levels.

### high
The editor will want to act on this. Includes:
- First quantification of something previously only hyped
- A previously-named partner has been **removed**
- Guidance changed (in either direction)
- A new risk factor added or an existing one materially intensified
- Strategy pivot (a previously stated direction has been replaced with another)
- A pilot has been productionised (`deployment` claim now appears where previously only `strategy`)
- A productionised system has gone quiet (often a `removed` change — possibly a failed rollout)

### medium
Real change worth knowing but not a story on its own:
- KPI moved within a previously-disclosed metric
- Capex split clarified
- New vendor named for a previously-announced initiative
- Forward target extended in time or scope without changing direction
- Risk-factor language tightened or loosened without adding/removing a risk

### low
Real change but likely only relevant to model-builders / sell-side:
- Tax-rate guidance moved
- Minor segment reclassification or restatement
- Cosmetic reword of hype with no substance change
- Pension/lease accounting language changes

**The honesty test:** if you can't write a one-sentence pitch to an editor explaining why they'd care, the value isn't `high`.
