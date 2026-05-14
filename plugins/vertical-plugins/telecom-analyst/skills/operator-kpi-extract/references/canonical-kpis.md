# Canonical KPI Vocabulary

Master list of canonical KPI names used by `operator-kpi-extract`. Each KPI lists the canonical key, the unit, common disclosure synonyms, and any ANTA-specific convention.

**Use canonical keys verbatim.** If you find a KPI the operator discloses that isn't in this list and doesn't map to a canonical, capture it under a `custom_kpis` sub-object on the segment, with the raw label and a one-line explanation.

## Income statement

### `revenue_total`
- **Unit:** `millions` (always in the segment's reporting currency)
- **Definition:** Total reported revenue including service revenue and equipment revenue.
- **Synonyms:** "Total revenue", "Group revenue", "Net revenue", "Turnover" (UK)
- **YoY:** Often disclosed twice — reported (`yoy_pct`) and organic / underlying (`yoy_organic_pct`). Capture both when given.

### `revenue_service`
- **Unit:** `millions`
- **Definition:** Service revenue only (mobile + fixed services, excluding equipment / handset sales).
- **Synonyms:** "Service revenue", "Communications revenue"
- **YoY:** Capture both reported and organic when given.

### `revenue_mobile_service`
- **Unit:** `millions`
- **Definition:** Mobile service revenue only.
- **Synonyms:** "Mobile service revenue", "Wireless service revenue"

### `revenue_fixed_service`
- **Unit:** `millions`
- **Definition:** Fixed service revenue (broadband, voice, TV).
- **Synonyms:** "Fixed service revenue", "Wireline revenue", "Broadband and TV revenue"

### `revenue_b2b` / `revenue_b2c`
- **Unit:** `millions`
- **Definition:** Customer-segment splits if disclosed.
- **Synonyms:** "Enterprise revenue", "Consumer revenue", "Business revenue"

### `revenue_equipment`
- **Unit:** `millions`
- **Definition:** Equipment / handset sales revenue.
- **Synonyms:** "Equipment revenue", "Handset revenue"

### `ebitda`
- **Unit:** `millions`
- **Definition:** EBITDA before the impact of lease accounting (IFRS 16 leases as opex).
- **Synonyms:** "Reported EBITDA", "Adjusted EBITDA" (sometimes — careful, this can include further adjustments)
- **⚠ Awkward:** Many European telcos (Vodafone, Orange, BT, TI) disclose **EBITDAaL** but not EBITDA. In that case set `ebitda: null` and populate `ebitda_aL`. Do not derive one from the other unless the operator discloses lease impact separately.

### `ebitda_aL`
- **Unit:** `millions`
- **Definition:** EBITDA After Leases — EBITDA minus IFRS 16 lease depreciation and interest impact. Closer to pre-IFRS-16 EBITDA.
- **Synonyms:** "EBITDAaL", "EBITDA after leases", "Adjusted EBITDAaL"

### `ebitda_margin_pct` / `ebitda_aL_margin_pct`
- **Unit:** `pct`
- **Definition:** EBITDA (or EBITDAaL) as % of revenue_total.
- **YoY:** Movement in **bps** — use `yoy_delta_bps`, not `yoy_pct`.
- **Calculated:** If only the underlying values are disclosed, calculate and flag `calculated: true`.

### `net_income`
- **Unit:** `millions`
- **Definition:** Net income / profit for the period attributable to shareholders.
- **Synonyms:** "Profit for the period", "Net profit", "Profit attributable to owners"

## Cash flow / balance sheet

### `free_cash_flow`
- **Unit:** `millions`
- **Definition:** The operator's headline FCF metric.
- **Synonyms:** "Free cash flow", "FCF", "Adjusted FCF"
- **⚠ Awkward:** Operators disclose multiple FCF measures (equity FCF, operating FCF, FCF excluding spectrum, etc.). Capture the headline one in `free_cash_flow` and the others in the specific keys below.

### `equity_free_cash_flow`
- **Unit:** `millions`
- **Definition:** FCF available to equity holders after interest and tax.
- **Synonyms:** "Equity FCF", "Equity free cash flow"

### `operating_free_cash_flow`
- **Unit:** `millions`
- **Definition:** Operating FCF — typically EBITDA - capex.
- **Synonyms:** "OpFCF", "Operating FCF"

### `capex_total`
- **Unit:** `millions`
- **Definition:** Total capital expenditure including spectrum.
- **Synonyms:** "Capex", "Capital expenditure", "Investment in property, plant and equipment"

### `capex_excl_spectrum` / `spectrum_capex`
- **Unit:** `millions`
- **Definition:** Capex excluding spectrum payments / spectrum capex only. Split out if disclosed separately.

### `capex_intensity_pct`
- **Unit:** `pct`
- **Definition:** `capex_total` / `revenue_total` × 100.
- **Calculated:** Usually calculated; some operators disclose it directly. Prefer disclosed over calculated.

### `net_debt`
- **Unit:** `millions`
- **Definition:** Net debt — gross debt minus cash. Use the operator's reported figure.
- **Synonyms:** "Net financial debt", "Net debt position"

### `leverage_x`
- **Unit:** `x` (multiple)
- **Definition:** Net debt / EBITDA (or EBITDAaL — note which in the segment `notes`).
- **Synonyms:** "Leverage", "Net debt / EBITDA"
- **Calculated:** Prefer the operator's disclosed figure; calculate if not given.

## Subscribers

All subscriber KPIs in `millions` unit unless the operator is small enough to report in thousands — capture the unit explicitly.

### `mobile_subs_total`
- **Unit:** `millions` or `thousands`
- **Definition:** Total mobile subscribers / connections (postpaid + prepaid + IoT, per the operator's definition).
- **Synonyms:** "Mobile customers", "Mobile connections", "Wireless subscribers"
- **⚠ Awkward:** Some operators include IoT/M2M in this number, some exclude. Note in segment `notes` if you can tell.

### `mobile_subs_postpaid` / `mobile_subs_prepaid` / `mobile_subs_5g`
- **Unit:** `millions` or `thousands`
- **Synonyms:** "Postpaid customers", "Prepaid customers", "5G connections", "5G subscribers"

### `fixed_broadband_subs`
- **Unit:** `millions` or `thousands`
- **Definition:** Total fixed-broadband subscribers across all technologies.
- **Synonyms:** "Broadband customers", "Fixed-line broadband subscribers"

### `ftth_subs`
- **Unit:** `millions` or `thousands`
- **Definition:** FTTH / FTTP fibre-to-the-premises subscribers.
- **Synonyms:** "Fibre customers", "FTTH subscribers", "FTTP customers"

### `tv_subs`
- **Unit:** `millions` or `thousands`
- **Synonyms:** "IPTV subscribers", "Pay-TV customers"

### `convergent_customers`
- **Unit:** `millions` or `thousands`
- **Definition:** Customers buying both mobile and fixed services from the operator.
- **Synonyms:** "Converged customers", "Fixed-mobile convergent"

### `mobile_net_adds_postpaid` / `fixed_broadband_net_adds`
- **Unit:** `thousands` (usually)
- **Definition:** Net additions during the period.

## ARPU

ARPU's currency matters. Use a `unit` like `EUR_per_month`, `USD_per_month`, `GBP_per_month`, etc. **Always per-month** — convert quarterly disclosures (divide by 3) and flag `calculated: true`.

### `mobile_arpu_postpaid` / `mobile_arpu_prepaid` / `mobile_arpu_blended`
- **Unit:** e.g. `EUR_per_month`
- **Synonyms:** "Postpaid ARPU", "Prepaid ARPU", "Mobile ARPU"

### `fixed_arpu`
- **Unit:** e.g. `EUR_per_month`
- **Synonyms:** "Fixed ARPU", "Broadband ARPU" (capture separately if disclosed separately)

### `convergent_arpu`
- **Unit:** e.g. `EUR_per_month`
- **Synonyms:** "Convergent customer ARPU", "Multi-play ARPU"

## Churn

Churn must specify monthly vs annualised in `unit`.

### `mobile_postpaid_churn_pct`
- **Unit:** `pct_monthly` or `pct_annualised`
- **Synonyms:** "Postpaid churn", "Contract churn"
- **⚠ Awkward:** US operators report monthly; many European operators report annualised. Always capture the period explicitly. Don't convert silently — flag conversions.

### `mobile_prepaid_churn_pct`
- **Unit:** `pct_monthly` or `pct_annualised`

### `fixed_broadband_churn_pct`
- **Unit:** `pct_monthly` or `pct_annualised`

## Network footprint

### `pops_covered_5g` / `pops_covered_5g_sa`
- **Unit:** `pct` (typically % of population covered) or `millions` of pops
- **Synonyms:** "5G population coverage", "5G SA coverage", "Standalone 5G footprint"

### `ftth_passings`
- **Unit:** `millions` of homes
- **Definition:** Premises / homes passed by FTTH network, regardless of whether subscribed.
- **Synonyms:** "FTTH homes passed", "Fibre footprint", "Premises passed"

### `ftth_take_up_pct`
- **Unit:** `pct`
- **Definition:** `ftth_subs` / `ftth_passings` × 100. Calculate if not disclosed.

### `tower_count`
- **Unit:** `count`
- **Definition:** Number of cell sites / tower assets — capture per operator's framing. Note if shared with a tower co.

### `spectrum_holdings_mhz`
- **Unit:** `MHz`
- **Definition:** Total MHz held across all bands in a country. Rarely disclosed cleanly at group level; usually appears in regulatory or country-specific commentary.

## Operational

### `headcount`
- **Unit:** `count`
- **Definition:** Total employees. FTE if disclosed; otherwise headcount.

## Worked awkward cases

### EBITDA vs EBITDAaL across an operator group

Vodafone, Orange, BT discloses **EBITDAaL** as the headline. Verizon, T-Mobile US disclose EBITDA pre-IFRS-16. Same canonical key would be misleading — keep them distinct (`ebitda` vs `ebitda_aL`) and populate whichever is disclosed. Note in segment `notes` which framework the operator uses.

### Organic vs reported YoY

European operators (Vodafone, Telefónica, Orange) almost always disclose **organic** growth that adjusts for FX, M&A, and disposals. The reported number reflects the underlying portfolio; the organic number reflects the underlying trajectory. **Both are interesting.** Capture both — `yoy_pct` for reported, `yoy_organic_pct` for organic. Don't pick one.

### Quarterly ARPU disclosure

US operators often disclose ARPU as a quarterly figure (e.g. "ARPA of $150 in Q3" — that's revenue per account per quarter). To get monthly-equivalent, divide by 3 and flag `calculated: true`. Capture the raw figure in `raw` so the conversion is traceable.

### Subscribers — connections vs customers

Some operators report connections (one customer can have multiple SIMs); some report customers (deduplicated). The numbers can differ materially. Capture what's disclosed under `mobile_subs_total` and note in segment `notes` which definition the operator uses if knowable.

### Currency mismatches across segments

A typical European operator: group reports in EUR; UK segment reports in GBP; US segment reports in USD. Each segment snapshot's `currency` field should be the segment's native disclosure currency, **not** the group's reporting currency. Don't convert silently.

### Restated prior periods

If the current doc presents prior-period numbers that differ from what was reported in the prior cycle's filing, that's a restatement. Capture in the top-level `restatements` array — old value, new restated value, the reason quote. This is important for the cycle_snapshots loader to handle correctly.
