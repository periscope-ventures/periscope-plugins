# Audit checks reference

What each check in `fin_qa_audit_company_facts` means, how seriously to take it, and how prone it is to false positives. Statuses are `ok`, `warn`, `problem`, `not_applicable`.

## ratio_inputs_resolve
**Means:** one or more standard concepts that ratios consume resolve to nothing (no filed tag, no derived formula), so those ratios are blank.
**Severity:** high, almost never a false positive. This is your list of blank-ratio causes.
**Fix:** map the tag if the company files it under a name that routes to null (EPS is the classic), or derive it if the company never files it but files the inputs (Liabilities, GrossProfit). See patterns.md.

## no_polluted_buckets
**Means:** a concept "bucket" (e.g. Revenue, CommonEquity, ShortTermDebt) has a component/disclosure tag summed in alongside the bucket's real total.
**Severity:** high when the polluter is a disclosure/footnote/contra line or a redundant total; **but frequently over-flags** legitimately additive components.
**How to tell which:** pull the actual bucket value (`fin_company_financial_line_item_over_time`) and compare to the filed balance sheet.
- If the bucket is ~2× the real line → genuine double-count, exclude the polluter.
- If the bucket equals the sum of genuinely separate balance-sheet lines (finance leases + notes + borrowings, all interest-bearing) → false positive, leave it.
Common genuine polluters: `TreasuryStockCommonValue` in CommonEquity, `InventoryValuationReserves` in Inventories, `AllowanceForDoubtfulAccountsReceivable` in TradeReceivables, debt-maturity-schedule tags (`...MaturitiesRepaymentsOfPrincipalInYear...`) in Long/ShortTermDebt, `InterestPaid` (cash-flow) in InterestExpense, `dei:EntityCommonStockSharesOutstanding` (cover page) in SharesYearEnd.

## no_duplicate_bucket_tags
**Means:** two tags in one bucket are both valued in the same period — a latent double-count *if* the bucket sums its tags.
**Severity:** medium, **highest false-positive rate of all the checks.** Often the two tags are (a) genuinely additive components, or (b) the same line under an old and new tag name that the engine already dedupes via period precedence.
**How to tell:** pull the number. If it matches the filed statement, the engine is handling it — false positive, optionally log. If it's doubled, fix. Watch especially for a redundant *total* (e.g. `us-gaap:LongTermDebt` duplicating `us-gaap:LongTermDebtNoncurrent`) → exclude the one with worse period coverage. But if excluding would blank early periods where it's the sole total, log an open issue instead.

## standardization_ambiguity
**Means:** a filed tag reverse-indexes to more than one standard concept (e.g. `us-gaap:CapitalLeaseObligations` → both ShortTermDebt and LongTermDebt; deferred-tax tags → current and noncurrent).
**Severity:** low-medium, a `warn`. It's a heads-up, not a confirmed defect. Most are harmless (the tag lands correctly in practice).
**Fix:** only act if a specific ratio reading that concept looks wrong. Then map the tag explicitly to the intended concept, or exclude it. Don't mass-fix every ambiguous tag.

## balance_sheet_equation
**Means:** checks Assets = Liabilities + Equity on the latest 10-K.
**Severity:** a `warn` for incomplete data is common and benign — it often means the company doesn't tag a standalone total `Liabilities` (many don't), so the equation can't be evaluated, not that it fails. Deriving Liabilities (see patterns.md) usually clears the incompleteness. An actual imbalance (equation evaluated and wrong) is high severity.

## income_statement_identity
**Means:** income-statement subtotals tie out (revenue − costs = operating income, etc.) across periods.
**Severity:** high when it fails. Usually `ok`. A failure points to a mis-mapped expense or subtotal tag.

## cash_flow_articulation
**Means:** Operating + Investing + Financing + FX = change in cash, across periods.
**Severity:** high when it fails. Usually `ok`. Failures often trace to a mid-year discrete-quarter de-cumulation gap (a known engine issue) — see the adjustment workflow in patterns.md for the manual-correction path.

## not_applicable statements (balance_sheet / income_statement / cash_flow)
If these three come back `not_applicable` with "the latest 10-K has no X statement," the company's most recent statements just aren't onboarded/cached yet — not a data defect. The structured facts (and thus most ratios) can still be fine. If the user needs those checks, onboard filings first: `onboard_facts_and_filings(cik, start_date=...)` or `ops_ingest_filing(cik, accessions=[...])`, then re-audit. Don't treat this as something to "fix" with mappings/excludes.

## Reading the summary
The tail `summary` gives counts of ok/problems/warnings. Target: drive `problems` down to only the confirmed false-positives, and understand every remaining `warn`. Zero is not the goal — *understood* is the goal.
