---
description: Draft the daily TelecomTV editorial briefing — overnight developments across the ANTA universe, today's calendar, notable absences, editorial actions
argument-hint: "[region scope, optional, e.g. 'EMEA only'] [lookback window, optional, default '24 hours']"
---

# Morning Note Command

Generate today's TelecomTV editorial morning briefing. Anchored in overnight ANTA Supabase ingests + filings + the catalyst calendar.

## Workflow

### Step 1: Parse the inputs

- **Region scope** — optional; default global. Common alternatives: "EMEA only", "APAC only", "US + Canada only".
- **Lookback window** — optional; default 24 hours. Extend for Mondays / post-holiday catch-up.
- **Universe scope** — optional; default ANTA tier-1 + tier-2 operators + tracked vendors.

### Step 2: Verify and load

1. Confirm today's date.
2. Pull fresh ingests from `anta-supabase` in the lookback window (`source_docs`, `ai_mentions`, `ai_capex`, `vendor_mentions`, `earnings_call_themes`, `telecomtv_angles`, `news_items`).
3. Scan `filings-store` MCP for IR-page releases not yet in `source_docs`.
4. Pull today's events + this week's calendar (use `catalyst-calendar` output if loaded; build fresh if not).
5. Pull recent `/earnings-preview` outputs to identify prior-cycle threads coming due.

### Step 3: Invoke the skill

Use `skill: "morning-note"` to assemble the briefing.

### Step 4: Deliver output

Provide:

1. **Markdown morning-note document** with the fixed structure: top story, universe section (operators / vendors / regulatory), today's calendar, this-week watch, notable absences, editorial actions.
2. **Optional dispatch summary** — one-paragraph version of the top story + top editorial action, ready to paste into Slack or email. Offer this if the user asks for distribution-ready output.

### Step 5: Offer next steps

After delivering, offer:
- "Want me to commission `/tv-angle` for [filer]'s [filing]?"
- "Want me to draft the `/earnings-preview` for [filer]'s call this week?"
- "Want me to update the catalyst calendar to reflect any newly-announced dates from overnight?"

## Quality Checklist

Before delivery, confirm:
- [ ] Date is today (verified, not from training data)
- [ ] Top story is specific, anchored to a named filer/event, and materially new
- [ ] Every universe bullet has development + editorial take (not just news)
- [ ] No filer is mentioned without a development
- [ ] Today's calendar grounded in actual catalyst data (not invented)
- [ ] Notable absences honestly assessed — empty is better than padded
- [ ] Editorial actions are specific and actionable (named filer + specific action)
- [ ] Length ≤ 1 page when rendered
- [ ] No equity-research artefacts (trade ideas, rating changes, price targets)
