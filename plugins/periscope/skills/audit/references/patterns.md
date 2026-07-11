# Fix patterns

Concrete recipes for the common findings, with real examples. Every example uses bare-digit `cik` and a real `reason` string. Adapt the tags/concepts to the company in front of you — these are the shapes, not a fixed script.

## Pattern A — Map a tag that resolves to null

**Symptom:** `ratio_inputs_resolve` says a concept is unresolved; `resolve_for_company` shows the company files a plausible tag but it maps to `null`.

**Classic case — EPS:** `us-gaap:EarningsPerShareBasic`/`Diluted` often resolve to null by default, blanking `eps_basic`/`eps_diluted`.

```
fin_qa_add_mapping(
  cik="1018724",
  tag="us-gaap:EarningsPerShareBasic",
  concept="EarningsPerShareBasic",
  reason="Company files us-gaap:EarningsPerShareBasic but default reverse-index resolves it to null, leaving eps_basic blank.")
```

Same for diluted. Verify: pull `per_share` ratios; EPS should now be the reported per-share figure.

## Pattern B — Derive a total the company never files

**Symptom:** `ratio_inputs_resolve` says a concept is unresolved, and `resolve_for_company` shows the company files the *inputs* but no tag for the total itself.

**Classic case — Liabilities:** many companies never tag a standalone total `us-gaap:Liabilities`; they file total-liabilities-and-equity and total equity. Derive it.

```
fin_qa_add_derived(
  cik="1018724",
  concept="Liabilities",
  op="subtract",
  inputs=["LiabilitiesAndEquity", "CommonEquity"],
  reason="Company never tags a total Liabilities line. Total liabilities = LiabilitiesAndEquity minus total equity (CommonEquity holds total stockholders' equity here).")
```

**Classic case — GrossProfit:** files Revenue and Cost of Revenue, no GrossProfit line, so gross_margin is blank.

```
fin_qa_add_derived(
  cik="1018724",
  concept="GrossProfit",
  op="subtract",
  inputs=["Revenue", "CostOfGoodsAndServicesSold"],
  reason="Company files Revenue and CostOfGoodsAndServicesSold but never tags GrossProfit, so gross_margin is blank.")
```

**Critical:** `inputs` are **standard** concept names, resolved through this company's *own* mappings. So if you remapped equity into `CommonEquity`, use `CommonEquity` in the derivation — not the raw tag, and not a concept that's now empty. A filed value for the derived concept always wins, so deriving never overwrites a real number — it only fills gaps.

## Pattern C — Exclude a disclosure/contra tag polluting a total

**Symptom:** `no_polluted_buckets` flags a bucket; `resolve_for_company` shows a footnote/disclosure/contra tag landing in the same concept as the real total; the bucket value is inflated vs the filing.

**Examples (all the same shape):**
- `us-gaap:TreasuryStockCommonValue` polluting `CommonEquity` (contra-equity, not common stock).
- `us-gaap:InventoryValuationReserves` polluting `Inventories` (reserve disclosure, not net inventory).
- `us-gaap:AllowanceForDoubtfulAccountsReceivable` polluting `TradeReceivables` (contra-asset).
- `us-gaap:InterestPaid` polluting `InterestExpense` (cash-flow supplemental, not IS expense).
- `us-gaap:LongTermDebtMaturitiesRepaymentsOfPrincipalInYear{Two..Five,AfterYearFive}` and `...InNextTwelveMonths` polluting Long/ShortTermDebt (maturity schedule, sums to ~total debt).
- `dei:EntityCommonStockSharesOutstanding` polluting `SharesYearEnd` (cover-page count, duplicates the GAAP balance-sheet count).
- `us-gaap:FinanceLeaseLiabilityUndiscountedExcessAmount` polluting `LongTermDebt` (imputed-interest excess, not principal).

```
fin_qa_add_exclude(
  cik="1018724",
  tag="us-gaap:TreasuryStockCommonValue",
  reason="Contra-equity treasury line reverse-indexes into CommonEquity, polluting the common-equity total. Not part of common stock par value.")
```

## Pattern D — Exclude a redundant total that double-counts

**Symptom:** `no_duplicate_bucket_tags` flags two tags in one bucket, and pulling the number shows the bucket ≈ 2× the filed line.

**Classic case:** `us-gaap:LongTermDebt` and `us-gaap:LongTermDebtNoncurrent` both carry the balance-sheet "long-term debt, excluding current" line and both route to `LongTermDebt`, roughly doubling it.

**Before excluding, check period coverage** (`fin_company_financial_line_item_over_time` with ~40 periods). Keep the tag with complete, correct coverage; exclude the redundant one.

```
fin_qa_add_exclude(
  cik="1018724",
  tag="us-gaap:LongTermDebt",
  reason="us-gaap:LongTermDebt and us-gaap:LongTermDebtNoncurrent both carry the same balance-sheet line and both route to LongTermDebt, ~doubling the bucket. LongTermDebtNoncurrent has full coverage and matches the filing; exclude the redundant LongTermDebt tag.")
```

**If the redundant tag is the *only* total in some earlier periods**, DON'T exclude globally — that blanks those periods. Log an open issue instead (Pattern F).

## Pattern E — Remap a concept + exclude the small line it now duplicates

**Symptom:** an equity/total ratio is wildly off (debt-to-equity in the thousands, book value ~$0.01/share) because the ratio's concept only captured a tiny component.

**Classic case — total equity:** ratios key off `CommonEquity`, but by default only `us-gaap:CommonStockValue` (par value, a few $M) routes there. Real total equity is `us-gaap:StockholdersEquity` (routes to `AllEquityBalance` by default). Two-step fix:

```
# 1. point total equity at CommonEquity
fin_qa_add_mapping(
  cik="1018724", tag="us-gaap:StockholdersEquity", concept="CommonEquity",
  reason="Leverage/per-share ratios key off CommonEquity but only par-value CommonStockValue routed there, giving absurd ratios. Map total StockholdersEquity to CommonEquity.")

# 2. exclude the par-value line so it doesn't sum on top of total equity
fin_qa_add_exclude(
  cik="1018724", tag="us-gaap:CommonStockValue",
  reason="Now that total StockholdersEquity feeds CommonEquity, the par-value CommonStockValue would sum on top, double-counting. Exclude it.")
```

**Cascade watch:** this remap empties `AllEquityBalance`. Any derivation using `AllEquityBalance` (like a derived Liabilities) must be repointed to `CommonEquity`. Re-verify immediately after.

## Pattern F — Log an open issue (deliberately not force-fixing)

**When:** the fix needs period-scoped logic the mapping/exclude tools can't express, or a filed-but-sparse value blocks a derivation.

**Classic case:** ASC 606 revenue transition — both `us-gaap:SalesRevenueNet` (legacy) and `us-gaap:RevenueFromContractWithCustomerExcludingAssessedTax` (modern) exist in overlap years, but the legacy tag is the *sole* revenue total in earlier years, so you can't globally exclude it.

```
fin_qa_add_open_issue(
  cik="1018724",
  concept="Revenue",
  tier=2,
  issue="Revenue bucket holds two total tags in ASC606 transition periods; engine precedence currently picks one correctly (verified vs filing), but it's a latent risk.",
  reason="Not fixed via exclude because SalesRevenueNet is the SOLE revenue total in earlier years; excluding blanks that history. Needs period-scoped precedence (tier-2 engine work).")
```

**Tiers:** 1 = per-company override could fix it (but you're choosing not to now), 2 = calc-engine change needed, 3 = upstream ingestion gap.

## Pattern G — Manual adjustment for a sourced blank/wrong cell (last resort)

**When:** a specific cell is blank/wrong due to an engine gap (classic: a mid-year investing/financing cash-flow discrete quarter dropped by de-cumulation of a negative), no scoped override fixes it, and you can source the correct number from filings (YTD minus prior YTD).

**Required order:** log the issue first, then post the adjustment resolving it.

```
# 1. log
fin_qa_add_open_issue(cik=..., concept="NetCashFromInvestingActivities", tier=2,
  issue="Q3 discrete investing-CF null: engine drops de-cumulation of negative quarters.",
  reason="Correct value is cross-accession YTD subtraction; resolving via adjustment.")
# -> returns an issue id

# 2. post the sourced value (verify by re-pulling the statement)
fin_qa_add_adjustment(cik=..., concept="NetCashFromInvestingActivities",
  period_end="2025-09-30", value=-21848000000, accession="0001628280-25-047240",
  reason="Q3 = YTD-9M minus YTD-6M across two accessions.", resolves_issue=<id>)
```

`concept` must be the **standard** concept name (use `resolve_for_company` to confirm), not the raw us-gaap tag.

## Verification snippets

After any fix batch, pull and eyeball:
- Leverage/per-share: `fin_company_financial_ratios_over_time(identifier, groups=["leverage","per_share"], periods=4, show_quarters=true, show_years=false, ...)` — book value should be tens of dollars, debt/equity order-of-magnitude ~1, equity multiplier ~2 for a typical large-cap.
- The corrected line item: `fin_company_financial_line_item_over_time(identifier, concepts=[...])` — compare to the filed statement.
- Whole balance sheet tie-out: `fin_company_financials_over_time(identifier, statements=["balance_sheet"], periods=1)` — Assets should equal Liabilities+Equity, and derived/excluded concepts should show the right standardized values (excluded tags show `standard_concept: null`).

Internal-consistency identities that must hold: `equity_multiplier = 1 + debt_to_equity` (total-liabilities basis); `debt_to_assets × equity_multiplier = debt_to_equity`.
