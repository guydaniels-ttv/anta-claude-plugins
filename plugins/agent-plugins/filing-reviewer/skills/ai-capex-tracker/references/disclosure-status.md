# Disclosure Status — Worked Examples

Reference for the four `disclosure_status` values. The `committed` vs `announced` distinction is the most common judgement call.

## spent

Past, realised, in the financials. Often retrospective. Tense is the giveaway.

| Quote | Status | Why |
|---|---|---|
| "We deployed $50m of AI infrastructure capex this quarter." | **spent** | Past tense, single-period, scoped. |
| "AI-related opex of €15m in H1 2026, primarily cloud consumption for our GenAI rollout." | **spent** | Realised, period stated. |
| "Of our $400m total Q3 capex, approximately $60m went to data-centre buildout for AI workloads." | **spent** | Realised share of realised total. |
| "Cumulative AI investment to date stands at $200m." | **spent** | Past, cumulative — set `horizon_type: multi-year-cumulative`. |

## committed

Contractually committed. Signed contracts, multi-year purchase agreements, binding orders. Not yet spent, but bound.

| Quote | Status | Why |
|---|---|---|
| "We have committed $300m under our multi-year GPU supply agreement with NVIDIA." | **committed** | Signed agreement, supplier named. |
| "Our POs in place for AI infrastructure total $120m for FY26." | **committed** | Purchase orders are binding. |
| "Under contract for $80m of AI compute capacity through 2027." | **committed** | Contractual language. |

## announced

Publicly announced plan with a horizon. Has shape (number, period, scope) but not yet contractually bound.

| Quote | Status | Why |
|---|---|---|
| "We plan to invest $500m in AI infrastructure over the next three years." | **announced** | Plan, horizon, amount — no contracts cited. |
| "By FY28 our cumulative AI capex will reach $1bn." | **announced** | Forward target with a date. |
| "Our three-year programme targets $750m of AI-related investment." | **announced** | Programme stated, target stated. |

## aspirational

Vague target, no timeline, or so loose as to be promotional rather than informational.

| Quote | Status | Why |
|---|---|---|
| "We expect to invest significantly in AI over the coming years." | **aspirational** | No number, no timeline. |
| "AI is and will remain a major investment area for the group." | **aspirational** | No number, no timeline. |
| "Meaningful capex allocated to our AI agenda." | **aspirational** | "Meaningful" is not a number. |
| "Materially higher AI spend in FY27 vs FY26." | **aspirational** | Direction stated, but no number — capture for the trajectory note, but classify as aspirational. |

**Why this matters:** an `aspirational` mention should never roll up into a tracked AI-capex number. Treat it as colour commentary. It's still worth capturing for the editorial summary, but downstream consumers must be able to filter it out.

## committed vs announced — the hard call

Look for explicit contractual language:
- "committed," "under contract," "POs in place," "binding orders," "signed agreements totalling," "purchase commitments of" → **committed**
- "plan to," "intend to," "by [date] we will," "our programme will," "targets," "expects" → **announced**

When in doubt → **announced**. Under-classifying as `announced` is safer than over-claiming `committed`.

A multi-year programme with a signed first-year contract is **`committed` for the first year, `announced` for the remainder.** Two separate disclosures, not one.

## Hedged numbers — "up to" / "in the range of" / "around"

Don't pretend a hedged number is firm. The skill captures the disclosure as it was made.

| Quote | How to capture |
|---|---|
| "Up to $500m over the next three years." | `amount_normalized: 500000000`, flag `notes: 'capped — "up to" language; actual may be lower'`. |
| "In the range of $400-500m." | `amount_normalized: 450000000` (midpoint), `notes: 'range $400-500m; midpoint used'`. Alternative: keep `amount_normalized: 500000000` with `notes` clear — pick midpoint as the default. |
| "Around $300m." | `amount_normalized: 300000000`, `notes: 'approximate'`. |
| "At least $100m." | `amount_normalized: 100000000`, `notes: 'floor disclosed only; actual may be higher'`. |
| "Approximately $50m, give or take 10%." | `amount_normalized: 50000000`, `notes: 'approximate, ±10%'`. |

## Currency, units, and notation

Normalize to integers in the major currency unit (no decimal places for whole-million amounts):

- "$500m" → `amount_normalized: 500000000`, `currency: "USD"`
- "€1.2bn" → `amount_normalized: 1200000000`, `currency: "EUR"`
- "INR 500 cr" (Indian crores = 10 million) → `amount_normalized: 5000000000`, `currency: "INR"`
- "JPY 80bn" → `amount_normalized: 80000000000`, `currency: "JPY"`

If currency is unclear from the doc, set `currency: null` and capture the raw notation in `amount_raw`. Don't guess.

## Ratio to total capex

Compute `ratio_to_total_capex` only when:
- The AI disclosure and the total capex are in the **same currency**
- The AI disclosure and the total capex cover the **same horizon** (don't compare a Q3 spend to an FY guide)

Otherwise leave `ratio_to_total_capex: null` and note the mismatch.

`ratio_source: "disclosed"` when the doc itself states the ratio ("AI represents 15% of our FY26 capex"). `ratio_source: "calculated"` when you computed it from two separate disclosures in the same doc.

## Edge cases

### "Investment in AI talent" → opex or capex?

Hiring and training are opex. Even when framed as "investment," classify `capex_or_opex: opex`. Worth capturing because total AI-related spend (capex + opex) is editorially interesting, but the column matters.

### Cloud consumption for AI workloads

Cloud is opex unless the disclosure specifies a multi-year prepaid commitment (which sometimes gets capitalised). Default to **opex** unless the doc explicitly states otherwise.

### Data-centre buildout — AI capex or general capex?

Capture as AI capex only if the doc specifically links the data-centre to AI workloads ("data-centre expansion to support AI compute," "GPU-ready facility"). A generic data-centre mention is not AI capex.

### Acquisitions of AI companies

M&A consideration for AI acquisitions is editorially interesting. Capture in a disclosure row with `ai_scope: "specific use case"` (and the acquired company in `context_tags`) but flag clearly in `notes` that this is M&A consideration, not organic capex. The `disclosure_status` is typically `spent` (closed) or `committed` (signed, pending close).
