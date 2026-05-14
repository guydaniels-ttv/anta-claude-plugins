# Source-doc envelope convention

All telecom-analyst extraction skills that emit structured output for ingestion into the ANTA Supabase wrap their JSON output in a shared `source_doc` envelope at the top level. A loader can then UPSERT one row into the `source_docs` parent table per filing and link every child row across every skill to a single `source_doc_id`.

## When this applies

This convention is mandatory for any skill whose output is intended to land in one of the extraction tables that FK into `source_docs`:

- `ai-mentions-extractor` → `ai_mentions`
- `vendor-mentions` → `vendor_mentions` (+ `vendor_universe_candidates`)
- `ai-capex-tracker` → `ai_capex`
- `earnings-call-themes` → `earnings_call_themes` (table to be designed)
- `telecomtv-angle` → `telecomtv_angles` (table to be designed)

It does **not** apply to:

- Skills whose output isn't a per-filing extraction (e.g. `filing-diff` summarises across cycles, `operator-kpi-extract` produces operator-level snapshots that map to `cycle_snapshots`, narrative skills like `morning-note` don't produce DB rows).
- Skills whose output is anchored to a different parent (e.g. `peer-set` constructs a per-filer peer set; its rows FK into a `peer_set_runs` parent rather than `source_docs`. That skill defines its own `peer_set_run` envelope — see its `SKILL.md`).

When you author a new extraction skill that doesn't fit either pattern, prefer adding a sibling envelope (similar shape — `<parent_name>` block at the top, child rows below — but pointing at a different parent table) over forcing a square peg into the `source_doc` envelope.

## The envelope

```json
{
  "source_doc": {
    "filer_kind": "operator | vendor",
    "filer_name": "string — canonical name from the ANTA universe",
    "filer_operator_id": "uuid or null — populated when filer_kind=='operator' AND a match exists in operators table",
    "doc_type": "annual_report | 10-K | 20-F | quarterly_report | earnings_call_transcript | investor_day | press_release | investor_deck | analyst_day | cmd | regulatory_filing | other",
    "title": "string — human-readable doc title, e.g. 'VEON Q3 2026 earnings call transcript'",
    "source_url": "string or null",
    "filing_date": "YYYY-MM-DD or null — publication date of the doc",
    "reporting_period": "string — as stated by the filer, e.g. 'Q3 2026' or 'FY 2025'",
    "scoring_cycle_id": "uuid or null — the ANTA scoring cycle this run feeds; null for ad-hoc runs",
    "reporting_currency": "ISO 4217 or null — the filer's reporting currency, only required when amounts are extracted",
    "extracted_by": "string — skill name + version, e.g. 'ai-mentions-extractor v1'"
  },
  "<rows>": [ /* skill-specific child rows */ ],
  "summary": "string — 3 to 6 sentence editorial narrative"
}
```

Some skills add additional top-level keys alongside `source_doc` and the rows array (e.g. `ai-capex-tracker` adds `total_capex_disclosed`, `vendor-mentions` adds `universe_candidates`). The envelope is the *minimum* — skills can extend it.

## Field rules

**`filer_kind`** — `'operator'` if the filer is in the ANTA operator universe, `'vendor'` otherwise. Drives the DB CHECK constraint that operator filers must have an FK and vendor filers must not (until a vendors table exists).

**`filer_name`** — always populated, always canonical. For operator filers, this is the canonical name from the `operators` table. For vendor filers, use a clean canonical form (no marketing suffix, no parent-co disambiguators) — this is what the future `filer_vendor_id` FK will be backfilled from.

**`filer_operator_id`** — populated only when `filer_kind == 'operator'` AND the lookup against `anta-supabase` returned a match. Otherwise `null`. The DB has a CHECK that rejects rows where this is set for vendor filers.

**`doc_type`** — required for the UNIQUE constraint on `source_docs (filer_name, reporting_period, doc_type)`. Pick the most specific value; default to `'other'` only when no enum option fits.

**`reporting_period`** — the period the filer is reporting on, **not** the ANTA scoring cycle. Free-form text but conventional values are `'Q1 2026'`, `'FY 2025'`, `'H1 2026'`, `'9M 2026'`. Used as part of the UPSERT key.

**`scoring_cycle_id`** — UUID FK to `scoring_cycles.id`. Pass through from the user's request context if known; leave `null` for ad-hoc extraction runs not tied to a cycle.

**`reporting_currency`** — required for `ai-capex-tracker`, optional otherwise. ISO 4217.

**`extracted_by`** — for provenance. Skill name plus version string. Lets a future audit query group rows by extractor.

## Why this shape

A single filing typically produces 10–30 ai_mentions + 10–30 vendor_mentions + 0–5 ai_capex disclosures. Storing the source-doc metadata once on a parent row avoids 30× duplication across children, and the editorial query "show everything we extracted from this filing" becomes a one-line FK lookup.

The denormalised `filer_operator_id`, `filer_name`, `scoring_cycle_id`, `reporting_period` columns on each child table are deliberate — the most common query pattern is filtering children by filer + cycle, and the denormalisation lets that be a single index scan rather than a join through `source_docs`. The loader is responsible for keeping the denormalised values in sync with the parent.

## Loader contract (informational)

The loader is expected to:

1. UPSERT a `source_docs` row using `(filer_name, reporting_period, doc_type)` as the conflict key. Returns `source_doc_id`.
2. For each child row in the skill output, INSERT into the matching child table with `source_doc_id` set, and the denormalised columns (`filer_operator_id`, `filer_name`, `scoring_cycle_id`, `reporting_period`, `filer_side` where applicable) copied from the envelope.
3. Where a re-run produces fresh rows, the loader is responsible for deleting the prior child rows for the same `source_doc_id` before re-inserting (no MERGE / UPSERT semantics on the children — they are append-on-fresh-extraction).

The loader does not yet exist. When it's authored, it will follow the existing `dossier_to_*.py` + `push_*.sh` sibling-loader pattern in the ANTA pipeline repo.

## Seeing this convention in action

See the Output section of any of these skills for the full envelope-plus-rows JSON shape:

- `skills/ai-mentions-extractor/SKILL.md`
- `skills/vendor-mentions/SKILL.md`
- `skills/ai-capex-tracker/SKILL.md`
