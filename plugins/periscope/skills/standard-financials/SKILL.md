---
name: standard-financials
description: The Periscope standard-concept catalog behind the standardized financials & ratios report — the global vocabulary each company's messy XBRL tags are mapped onto so periods and companies line up. Use this skill whenever you need to know what standardized line items exist, what a given concept means, which group/statement/section a line sits in, how a calculated concept or ratio is computed, or what standard_concept / group value to pass to fin_standard_financials_and_ratios (PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS) or to filter fin_company_financials_over_time_items. Covers the five groups (income statement, balance sheet, cash flow, disclosure, ratios), every concept's label + description + notes, the six intrinsic-valuation ratios, and the Damodaran FCFF/FCFE valuation chain (tax rate → NOPAT → reinvestment → FCFF/ROIC → growth). Triggers: "what does ROIC/NOPAT/FCFF mean here", "list the standardized balance-sheet line items", "which standard concept is revenue", "how is InvestedCapital calculated", "what group is Goodwill in".
---

# Standard financials & ratios — the concept catalog

Periscope's **standardized** financials are built on one global vocabulary: the
**standard-concept catalog**. Every company's raw, idiosyncratic XBRL tags are mapped onto
these concepts per filing, so `fin_standard_financials_and_ratios`
(`PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS`) can render a clean, comparable statement grid
where the same line means the same thing across companies and periods.

This skill is the reference for that catalog. The complete, grouped list — every concept's
id, line-item label, description, formula, and valuation role — is in
[references/concepts.md](references/concepts.md). Read it before answering anything about a
specific line item, its meaning, its group, or how a calculated value/ratio is derived.

The catalog is scoped for **Damodaran intrinsic valuation**: the filed statement + lease/
option disclosures are the raw material, and the calculated concepts are the FCFF/FCFE
chain (effective tax rate → NOPAT → reinvestment → FCFF / ROIC → expected growth) plus his
R&D and operating-lease capitalization adjustments.

## Two axes describe every concept

1. **Group** (`group`) — its *home*: `income_statement`, `balance_sheet`, `cash_flow`,
   `disclosure` (footnote-sourced: leases, options), or `ratios`. These are exactly the
   values the `group` argument narrows to.
2. **Filed vs calculated** (`is_calculated`) — separate from group. A **filed** concept is
   bound to raw facts per filing; a **calculated** concept is computed from other concepts
   via a formula. A calculated aggregate lives under the statement it belongs to (`EBITDA`
   under the income statement, `FCFF` under cash flow); only the **six ratios** live under
   `ratios`.

Within a group, concepts are ordered by **section** (e.g. current assets → non-current
assets → … on the balance sheet), then catalog order — so the report reads top-to-bottom
like a real filing. A `section` shown in the catalog is also what
`fin_company_financials_over_time` reports on. Some concepts are **subtotals**
(`GrossProfit`, `TotalAssets`, `CashFromOperations`, `NetIncome`, …) — a reader indents the
detail lines under them.

### Filed-preferred hybrids

`GrossProfit` and `OperatingIncome` are **filed-preferred**: mappable like any filed line,
but *synthesized* from their components when the filer omits the subtotal (e.g. IBM prints
no EBIT line). The filed value always wins when present; the fallback only fills a gap.

## The six intrinsic-valuation ratios

Only these live under `ratios` (all calculated):

| Ratio | Formula | Measures |
| :-- | :-- | :-- |
| `EffectiveTaxRate` | `IncomeTaxExpense / PretaxIncome` | tax drag on operating income |
| `ROIC` | `NOPAT / InvestedCapital` (capital lagged 1 period) | quality of growth |
| `SalesToCapital` | `Revenue / InvestedCapital` | capital efficiency of revenue |
| `ReinvestmentRate` | `Reinvestment / NOPAT` | share of NOPAT plowed back |
| `ExpectedGrowth` | `ReinvestmentRate × ROIC` | fundamental operating-income growth |
| `InterestCoverage` | `OperatingIncome / InterestExpense` | synthetic credit rating → cost of debt |

## How the catalog reaches a spreadsheet / a tool call

- **Whole standardized report:** `fin_standard_financials_and_ratios(identifier)` /
  `=PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS("AAPL")`. Narrow with `group=` to one of the
  five groups above. Each row carries its label, a "Formula / notes" column, and one value
  per period. (See the **prsc-functions** skill for the Excel-formula mechanics.)
- **Filter as-filed items to a standard concept:** pass a catalog id as `standard_concept`
  to `fin_company_financials_over_time_items` (e.g. `standard_concept="Revenue"`), or set
  `reveal_mapping=TRUE` on `fin_company_financials_over_time` to see which catalog concept
  each raw row feeds.
- Use the **exact concept id** from the catalog (PascalCase, e.g. `OperatingIncome`) where
  a tool takes `standard_concept`, and the **group value** (snake_case, e.g.
  `income_statement`) where it takes `group`.

## Mapping is per-filing — coverage varies

The catalog is the *only* global artifact. **Which raw XBRL fact feeds each concept is
authored per `(cik, accession)`** and never assumed to transfer across companies. So:

- A standardized row is only populated if that filing has a mapping authored for it. An
  unmapped or thinly-mapped filer returns sparse rows — that's missing authoring, **not** a
  formula error. Fall back to `fin_company_financials_over_time` (as-filed) for the raw facts.
- A calculated concept/ratio is only as complete as its inputs; a null cell usually traces
  to a missing input concept.
- Use `fin_reconcile` to check a company's mapping coverage and that the accounting
  identities tie (assets = liabilities + equity, the CFO reconciliation, etc.).

## Answering catalog questions

1. Identify the line item / concept the user means (by label or by id) in
   [references/concepts.md](references/concepts.md).
2. Give its **id**, **group**, **description**, and — if calculated — its **formula**; add
   the **valuation role** when relevant (many concepts note their Damodaran role).
3. If they're heading for a tool call or Excel formula, hand them the exact
   `group=` / `standard_concept=` value (and point to the **prsc-functions** skill for the
   `=PRSC.*` form).
