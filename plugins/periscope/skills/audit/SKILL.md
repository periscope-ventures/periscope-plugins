---
name: audit
description: Audit and repair a company's financial data quality in Periscope. Use this skill whenever the user asks to "audit" a company's financials, run a data-quality check, diagnose why a ratio or line item is blank/wrong/absurd, or clean up XBRL tag mapping issues for a ticker or CIK. Trigger it for any request like "audit AMZN", "why is debt-to-equity broken for this company", "fix the financials for X", "check TSLA's data quality", or "run QA on the pipeline for this filer" — even if the user doesn't say the word "audit". This is the go-to workflow for the Periscope fin_qa_* tools.
---

# Financial Data-Quality Audit & Repair

Audit a company's financial facts in Periscope, diagnose every problem, apply the narrowest safe fix, verify against the actual statements, and log anything that can't be safely auto-fixed. The output is a clean set of ratios/statements that tie to the company's real SEC filings, plus a short written report of what was changed and why.

The whole reason this skill exists: the raw audit output is long and noisy, many "problems" are false positives, and fixes **cascade** — a mapping change can empty out a derived formula's input, or expose a second problem that was masked by the first. Working the findings blindly, or one-at-a-time without re-verifying, produces confidently-wrong numbers. This skill encodes the diagnose → fix → re-verify loop that avoids that.

## The tools

All tools come from the bundled Periscope MCP connector — the `fin_qa_*` write/diagnose tools plus the `fin_*` read tools. Identify the company by ticker or CIK.

Read / diagnose:
- `fin_qa_audit_company_facts(identifier)` — the battery of checks. Start here.
- `fin_qa_resolve_for_company(identifier)` — the X-ray: shows what standard concept **every** raw tag resolves to, and via which layer (`reverse-index` = default, `custom-add` = your mapping, `custom-exclude` = your exclude). This is how you find the culprit tag behind a broken bucket.
- `fin_qa_get_data_quality(cik)` — shows the custom record as authored (mappings, excludes, derived, open issues, adjustments).
- `fin_company_financials_over_time`, `fin_company_financial_line_item_over_time`, `fin_company_financial_ratios_over_time` — the actual numbers. Use these to confirm a fix worked and didn't break a neighbor.

Write / fix (each is a scalpel — one entry, leaves the rest of the record untouched):
- `fin_qa_add_mapping(cik, tag, concept, reason)` — point a raw tag at the standard concept a ratio wants.
- `fin_qa_add_exclude(cik, tag, reason)` — drop a raw tag so it resolves to nothing.
- `fin_qa_add_derived(cik, concept, op, inputs, reason)` — define a concept the company never filed (`op` ∈ subtract/sum/divide; `inputs` are **standard** concept names, resolved through this company's own mappings).
- `fin_qa_add_open_issue(cik, concept, tier, issue, reason)` — log a finding you're deliberately not force-fixing.
- Removal counterparts: `fin_qa_remove_mapping/exclude/derived`, `fin_qa_resolve_open_issue`, `fin_qa_delete_data_quality` (nukes the whole record).

`cik` for write tools is bare digits (e.g. `1018724`), which `fin_lookup_company(ticker)` or the audit's returned `cik` gives you.

## Workflow

Do these in order. Don't skip the re-verify steps — that's where cascades get caught.

### 1. Audit and resolve

Run `fin_qa_audit_company_facts(identifier)`, then **immediately** run `fin_qa_resolve_for_company(identifier)` so you have the full tag→concept map in context before touching anything. The audit tells you *what's* wrong; the resolve map tells you *which tag* is causing it. You need both to fix safely.

Read `references/checks.md` for what each audit check means and which are prone to false positives.

### 2. Triage every finding

For each finding, classify it before acting. Read `references/patterns.md` — it has the concrete fix pattern for each of the common problems (unresolved ratio inputs, polluted buckets, duplicate tags, ambiguous standardization) with real examples. The classification you're making for each finding:

- **Genuine corruption → fix now.** A disclosure/footnote tag polluting a total (e.g. a treasury-stock line summed into common equity), a redundant total double-counting (two tags carrying the same balance-sheet line), a par-value line standing in for total equity. These produce visibly wrong numbers and have a clean scoped fix.
- **Blank because never filed → derive.** The company files the inputs but not the total (classic: files Revenue and Cost of Revenue but no GrossProfit; files total-liabilities-and-equity and equity but no standalone Liabilities). Derive it.
- **Blank because the tag resolves to null → map.** The company files the tag but the default doesn't route it (classic: `us-gaap:EarningsPerShareBasic` → null). Map it.
- **False positive → leave it, note it.** Legitimately additive components (finance leases + borrowings both belong in interest-bearing debt), or a redundant tag the engine already dedupes to the correct value via period precedence. Verify it's actually fine (pull the number), then move on.
- **Real but not safely auto-fixable → log an open issue.** Needs period-scoped logic the mapping/exclude tools can't express (e.g. "prefer tag A when both A and B are present, but A is the *only* total in earlier periods so you can't just exclude it"). Log it with the right tier.

### 3. Apply fixes — mind the ordering

Order matters because fixes chain. The dependency rule: **if you're going to remap the tag that a derivation depends on, fix the derivation's inputs too.** In the Amazon run, mapping `StockholdersEquity` from `AllEquityBalance` → `CommonEquity` silently emptied a derived `Liabilities = LiabilitiesAndEquity − AllEquityBalance`, because `AllEquityBalance` no longer had a value. The fix had to switch to `− CommonEquity`.

Practical sequencing:
1. Do the simple maps and excludes first (EPS mappings, disclosure-tag excludes).
2. When you remap a "total" concept (equity especially), immediately check whether anything derives from the old target, and whether a small component tag (par value) now needs excluding so it doesn't sum on top of the new total.
3. Do derivations last, once their input concepts are stable.

Every write needs a real `reason` — it's the audit trail. State what the tag actually is and why it doesn't belong / why the derivation is needed.

### 4. Re-verify after each cluster of fixes

After a batch, pull the affected ratios/line items and read the values. Two things to check:
- **Did the target fix land?** The ratio now populates and is realistic (Amazon book value went from $0.01 → $41/share).
- **Did anything break?** A neighbor ratio that was fine is now null or absurd → you triggered a cascade; trace it via `resolve_for_company` and fix the input.

Sanity-check internal consistency: `equity_multiplier = 1 + debt_to_equity` when both use total liabilities; `debt_to_assets × equity_multiplier = debt_to_equity`. If those identities don't hold, something's still wrong.

### 5. Re-audit

Run `fin_qa_audit_company_facts` again. Expect the problem count to drop but rarely to zero — the residual flags are usually the false-positives (additive components, period-precedence dedupes). Confirm each remaining flag is one you've consciously judged benign, not a real issue you missed.

### 6. Cross-check against reality (if asked, or for anything surprising)

The audit only checks internal consistency — it can't tell you the numbers match the actual company. For any figure you corrected that matters, or if the user asks, verify against the real SEC filing: `web_search` for the company's 10-K/10-Q balance sheet, or read it via the `fin_list_filings` → `fin_read_filing_section` tools. Match equity, revenue, net income, debt to the filed statements.

**Watch the fiscal-period labeling.** Periscope may label periods offset from the company's calendar fiscal year (Amazon's Dec-2025 year-end close showed as "FY 2025", the Mar-2026 quarter as "Q1 2026"). Discrete quarterly data is the reliable ground truth — sum four quarters to check against a reported annual. A mismatch in an FY-labeled column that disappears when you sum the quarters is a labeling artifact, not a data error.

### 7. Report

Summarize concisely: what was broken, what you changed (grouped as mappings / excludes / derivations / logged issues), and the before→after on the headline ratios. If you cross-checked, show the tie-out. Keep it tight — a table of the corrected metrics against the source is worth more than prose.

## Guardrails

- **Narrowest fix that works.** Prefer a single exclude/map over deleting the record. Never `fin_qa_delete_data_quality` unless the user explicitly wants to start over.
- **Don't over-exclude across periods.** A tag can be a redundant duplicate in recent periods but the *only* total in older ones (Amazon's `SalesRevenueNet` is redundant post-2016 but the sole revenue total 2009–2015). Excluding it globally blanks the early history. When a fix would be right for some periods and wrong for others, that's an open-issue (tier 2), not an exclude. Always check a tag's full period coverage (`fin_company_financial_line_item_over_time` with many periods) before a global exclude.
- **A filed value beats a derivation.** `fin_qa_add_derived` only fills gaps where nothing was filed. If a concept is filed-but-sparse (a `GrossProfit` tag present in some periods, blank in others), the derivation may not fill the blanks and you may need to log it rather than force it.
- **Believe the audit's structure, verify its severity.** The checks are conservative and over-flag. Treat every "problem" as a hypothesis to confirm by pulling the actual number, not as a confirmed defect.

## Common concept vocabulary

The standard concepts ratios key off (so you know what to map/derive toward): `Revenue`, `CostOfGoodsAndServicesSold`, `GrossProfit`, `OperatingIncomeLoss`, `NetIncome`, `Assets`, `Liabilities`, `CommonEquity` (total stockholders' equity for ratio purposes), `ShortTermDebt`, `LongTermDebt`, `InterestExpense`, `Inventories`, `TradeReceivables`, `SharesYearEnd`, `EarningsPerShareBasic`, `EarningsPerShareDiluted`, `CashAndCashEquivalents`, `CurrentAssetsTotal`, `CurrentLiabilitiesTotal`, `LiabilitiesAndEquity`. See `references/patterns.md` for how the messy real tags map onto these.
