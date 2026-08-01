# Standard-concept catalog

The complete global vocabulary behind `fin_standard_financials_and_ratios`
(`PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS`), grouped by **statement → section** in
presentation order. Concept ids are PascalCase (pass them as `standard_concept`); group
values are snake_case (pass them as `group`).

**Type legend:**
- **filed** — bound to raw XBRL facts per filing.
- **subtotal** — a face-statement subtotal/total (detail lines indent under it).
- **filed-preferred** — filed like any line, but synthesized from components when the filer
  omits it; the filed value wins when present (fallback formula shown).
- **calc** — calculated from other concepts (formula shown); not filed.
- **ratio** — one of the six intrinsic-valuation ratios (calculated; formula shown).
- **single** — filed, but takes exactly one leaf (non-additive: per-share / weighted averages).
- **memo** — filed but NOT rolled into the statement totals.

"Role" notes are the concept's part in Damodaran's FCFF/FCFE intrinsic valuation, where it
has one.

---

## Income statement — `group="income_statement"`

### Revenue → gross profit (`section: revenue`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `Revenue` | Revenue | filed | Net sales / total revenue for the period. |
| `CostOfRevenue` | Cost of revenue | filed | Cost of goods and services sold. |
| `GrossProfit` | Gross profit | filed-preferred, subtotal | Revenue minus cost of revenue. Fallback: `Revenue - CostOfRevenue` when the filer omits the subtotal. |

### Operating expenses → operating income (`section: operating_expense`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `ResearchAndDevelopment` | R&D expense | filed | Research & development expense. Role: capitalized into a research asset; adds back to adjusted operating income. |
| `SellingGeneralAndAdministrative` | SG&A | filed | Selling, general & administrative expense. |
| `OperatingIncome` | Operating income (EBIT) | filed-preferred, subtotal | Operating income / EBIT. Fallback: `GrossProfit - SG&A - R&D` when the filer prints no EBIT line. Role: base for after-tax operating income (NOPAT). |

### Non-operating → pretax (`section: non_operating`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `InterestExpense` | Interest expense | filed | Interest expense on debt. Role: interest coverage → synthetic rating → cost of debt. |
| `NonOperatingIncome` | Non-operating income | filed | Interest/investment and other non-operating income, net. |
| `PretaxIncome` | Pretax income | filed, subtotal | Income before income taxes. Role: denominator of the effective tax rate. |

### Taxes → net income (`section: bottom_line`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `IncomeTaxExpense` | Income tax expense | filed | Provision for income taxes. Role: numerator of the effective tax rate. |
| `NetIncome` | Net income | filed, subtotal | Net income (bottom line). Role: base for FCFE. |
| `MinorityInterestEarnings` | Net income to NCI | filed | Earnings attributable to non-controlling interests. |

### Per share (`section: per_share`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `EarningsPerShareBasic` | EPS (basic) | single | Basic earnings per share. |
| `EarningsPerShareDiluted` | EPS (diluted) | single | Diluted earnings per share. |
| `WeightedAvgSharesBasic` | Weighted-avg shares (basic) | filed | Weighted-average basic shares outstanding. |
| `WeightedAvgSharesDiluted` | Weighted-avg shares (diluted) | filed | Weighted-average diluted shares outstanding. |

### Supplemental analytical measures (`section: supplemental`)
| Concept | Line item | Type | Formula | Description & role |
| :-- | :-- | :-- | :-- | :-- |
| `EBITDA` | EBITDA | calc | `OperatingIncome + DepreciationAmortization` | Operating income plus D&A. |
| `NOPAT` | After-tax operating income | calc | `OperatingIncome * (1 - EffectiveTaxRate)` | EBIT after tax — Damodaran's EBIT(1−t). Role: numerator of ROIC; base of FCFF. |
| `AdjustedOperatingIncome` | Adjusted operating income | calc | `OperatingIncome + ResearchAndDevelopment - RDAmortization + imputed lease interest` | EBIT with R&D and operating leases capitalized. Role: cleaner EBIT for NOPAT and ROIC. |

---

## Balance sheet — `group="balance_sheet"`

### Current assets (`section: current_assets`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `CashAndCashEquivalents` | Cash & equivalents | filed | Cash and cash equivalents. |
| `ShortTermInvestments` | Short-term investments | filed | Marketable securities / short-term investments. Role: part of net debt. |
| `AccountsReceivable` | Accounts receivable | filed | Trade receivables, net. |
| `Inventory` | Inventory | filed | Inventories. |
| `OtherCurrentAssets` | Other current assets | filed | The filer's residual current-asset line (prepaid & other), mapped 1:1 — never a sweep of items that have their own concept. Role: non-cash working capital. |
| `TotalCurrentAssets` | Total current assets | filed, subtotal | Total current assets. |

### Non-current assets (`section: non_current_assets`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `PPENet` | PP&E (net) | filed | Property, plant & equipment, net of depreciation. |
| `Goodwill` | Goodwill | filed | Goodwill. |
| `IntangibleAssets` | Intangible assets | filed | Intangible assets excluding goodwill. |
| `LongTermInvestments` | Long-term investments | filed | Equity-method, non-marketable equity, and other long-term investments / cross-holdings. |
| `OperatingLeaseRightOfUseAsset` | Operating lease ROU asset | filed | Operating-lease right-of-use asset (ASC 842). Role: on-balance-sheet form of capitalized operating leases. |
| `OtherNonCurrentAssets` | Other non-current assets | filed | The filer's residual non-current-asset line, mapped 1:1. |
| `TotalAssets` | Total assets | filed, subtotal | Total assets. |

### Current liabilities (`section: current_liabilities`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `AccountsPayable` | Accounts payable | filed | Trade payables. Role: non-cash working capital. |
| `AccruedLiabilities` | Accrued & other current liabilities | filed | Accrued expenses and other current liabilities. Role: non-cash working capital. |
| `CurrentLeaseLiability` | Current lease liability | filed | Current portion of operating- and finance-lease liabilities. |
| `ShortTermDebt` | Short-term debt | filed | Short-term debt and current portion of long-term debt. |
| `DeferredRevenue` | Deferred revenue | filed | Current deferred revenue / contract liabilities. Role: non-cash working capital. |
| `IncomeTaxesPayable` | Income taxes payable | filed | Current income taxes payable. Role: non-cash working capital. |
| `OtherCurrentLiabilities` | Other current liabilities | filed | The filer's residual current-liability line, mapped 1:1. Role: non-cash working capital. |
| `TotalCurrentLiabilities` | Total current liabilities | filed, subtotal | Total current liabilities. |

### Non-current liabilities (`section: non_current_liabilities`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `LongTermDebt` | Long-term debt | filed | Long-term debt, non-current. |
| `OperatingLeaseLiability` | Operating lease liability (non-current) | filed | Non-current operating-lease liability (ASC 842); current portion is `CurrentLeaseLiability`. Role: added to debt in the adjusted capital structure. |
| `DeferredTaxLiabilities` | Deferred & long-term tax liabilities | filed | Deferred tax liabilities and other long-term income-tax liabilities. |
| `OtherNonCurrentLiabilities` | Other non-current liabilities | filed | The filer's residual non-current-liability line, mapped 1:1. |
| `TotalLiabilities` | Total liabilities | filed, subtotal | Total liabilities. |

### Equity (`section: equity`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `PaidInCapital` | Paid-in capital | filed | Common stock at par plus additional paid-in capital. |
| `RetainedEarnings` | Retained earnings | filed | Retained earnings / accumulated deficit (carries its filed sign — negative when a deficit). |
| `AccumulatedOCI` | Accumulated OCI | filed | Accumulated other comprehensive income / (loss) (carries its filed sign). |
| `CommonEquity` | Common equity | filed, subtotal | Total common shareholders' equity = paid-in capital + retained earnings + accumulated OCI. Role: book value of equity in invested capital. |
| `PreferredEquity` | Preferred equity | filed | Preferred stock / preferred equity. |
| `MinorityInterest` | Non-controlling interest | filed | Non-controlling (minority) interest on the balance sheet. Role: part of invested capital; netted in equity value per share. |
| `SharesOutstanding` | Shares outstanding | filed | Period-end common shares outstanding. Role: denominator of per-share value. |
| `TotalEquity` | Total equity | calc, subtotal | `CommonEquity + PreferredEquity + MinorityInterest` — total equity incl. preferred and NCI. |

### Capital structure & memo (`section: capital_structure`)
*Analytical aggregates and memos — off the face balance sheet, which ends at total equity with Assets = Liabilities + Equity.*
| Concept | Line item | Type | Formula | Description & role |
| :-- | :-- | :-- | :-- | :-- |
| `PPEGross` | PP&E (gross) | filed, memo | — | PP&E gross (PP&E-note figure; `PPENet` is the face line that rolls into total assets). |
| `RestrictedCash` | Restricted cash | filed, memo | — | Restricted cash (current + non-current). Embedded in other BS lines, so NOT added into asset roll-ups; exists so the cash-flow close ties (`CashAtEndOfPeriod == CashAndCashEquivalents + RestrictedCash`). |
| `ResearchAsset` | Capitalized R&D asset | calc | `capitalize(ResearchAndDevelopment history over amortizable life)` | R&D capitalized over an amortizable life. Role: added to invested capital; drives the R&D operating-income adjustment. |
| `TangibleCommonEquity` | Tangible common equity | calc | `CommonEquity - Goodwill - IntangibleAssets` | Common equity less goodwill and intangibles. |
| `TotalDebt` | Total debt | calc | `ShortTermDebt + LongTermDebt + OperatingLeaseLiability + CurrentLeaseLiability` | Short + long-term debt plus operating-lease liability (current + non-current). |
| `NetDebt` | Net debt | calc | `TotalDebt - CashAndCashEquivalents - ShortTermInvestments` | Total debt less cash and short-term investments. |
| `CapitalizedLeaseDebt` | Capitalized lease debt | calc | `OperatingLeaseLiability + CurrentLeaseLiability` (else `PV(OperatingLeaseFutureMinimumPayments)`) | Debt-equivalent of operating leases; PV of future commitments for pre-ASC842 filings. Role: added to debt and invested capital in the lease adjustment. |
| `InvestedCapital` | Invested capital | calc | `CommonEquity + MinorityInterest + TotalDebt - CashAndCashEquivalents - ShortTermInvestments` | Book equity + NCI + total debt − cash − ST investments. Role: denominator of ROIC and sales-to-capital. |
| `AdjustedInvestedCapital` | Adjusted invested capital | calc | `InvestedCapital + ResearchAsset + CapitalizedLeaseDebt` | Invested capital plus the research asset and capitalized leases. |
| `NonCashWorkingCapital` | Non-cash working capital | calc | `(TotalCurrentAssets - CashAndCashEquivalents - ShortTermInvestments) - (TotalCurrentLiabilities - ShortTermDebt)` | Non-cash current assets less non-debt current liabilities. |

---

## Cash flow — `group="cash_flow"`

### Operating activities (`section: operating_activities`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `DepreciationAmortization` | Depreciation & amortization | filed | D&A add-back in operating cash flow. Role: net cap-ex; EBITDA. |
| `StockBasedCompensation` | Stock-based compensation | filed | Stock-based compensation expense (CF add-back). |
| `DeferredIncomeTaxes` | Deferred income taxes | filed | Deferred income tax provision/benefit — a non-cash reconciling item. |
| `GainLossOnInvestments` | (Gain)/loss on investments | filed | Non-cash (gains)/losses on investments and securities, reversed out of net income. |
| `ImpairmentCharges` | Impairment charges | filed | Asset impairments and write-downs — a non-cash reconciling item. |
| `OtherNonCashItems` | Other non-cash items | filed | The filer's residual non-cash reconciling line, mapped 1:1. |
| `ChangeInAccountsReceivable` | Δ Accounts receivable | filed | Operating change in AR (an increase is a use of cash). |
| `ChangeInInventory` | Δ Inventory | filed | Operating change in inventory (an increase is a use of cash). |
| `ChangeInPrepaidAndOtherCurrentAssets` | Δ Prepaid & other current assets | filed | Operating change in prepaid expenses and other current assets. |
| `ChangeInOtherOperatingAssets` | Δ Other operating assets | filed | Operating change in other (largely non-current) operating assets. |
| `ChangeInAccountsPayable` | Δ Accounts payable | filed | Operating change in AP (an increase is a source of cash). |
| `ChangeInAccruedLiabilities` | Δ Accrued & other current liabilities | filed | Operating change in accrued expenses and other current liabilities. |
| `ChangeInOtherOperatingLiabilities` | Δ Other operating liabilities | filed | Operating change in other (largely non-current) operating liabilities. |
| `CashFromOperations` | Cash from operations | filed, subtotal | Net cash provided by operating activities. |

### Investing activities (`section: investing_activities`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `CapitalExpenditures` | Capital expenditures | filed | Purchases of PP&E. Role: reinvestment. |
| `PurchasesOfInvestments` | Purchases of investments | filed | Cash used to purchase marketable and non-marketable investments/securities. |
| `SalesOfInvestments` | Sales & maturities of investments | filed | Cash from sales and maturities of investments/securities. |
| `Acquisitions` | Acquisitions | filed | Cash paid for acquisitions, net. Role: inorganic reinvestment. |
| `OtherInvestingActivities` | Other investing activities | filed | The filer's residual investing line plus idiosyncratic investing flows with no standard concept. |
| `CashFromInvesting` | Cash from investing | filed, subtotal | Net cash provided by (used in) investing activities. |

### Financing activities (`section: financing_activities`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `DebtIssued` | Debt issued | filed | Proceeds from issuance of debt. Role: net borrowing (FCFE). |
| `DebtRepaid` | Debt repaid | filed | Repayments of debt. Role: net borrowing (FCFE). |
| `FinanceLeasePrincipalPayments` | Principal payments on finance leases | filed | Principal portion of finance-lease payments — kept apart from debt repayment. |
| `DividendsPaid` | Dividends paid | filed | Common dividends paid. Role: cash returned to equity (FCFE / payout). |
| `ShareRepurchases` | Share repurchases | filed | Cash paid to repurchase stock. Role: cash returned to equity (FCFE / payout). |
| `TaxesPaidOnNetShareSettlement` | Taxes paid on net share settlement | filed | Cash remitted to tax authorities on net-share-settled equity awards — kept apart from share repurchases. |
| `ProceedsFromStockIssuance` | Proceeds from stock issuance | filed | Cash from issuing stock — option exercises, ESPP, equity issuance. |
| `OtherFinancingActivities` | Other financing activities | filed | The filer's residual financing line plus idiosyncratic financing flows with no standard concept. |
| `CashFromFinancing` | Cash from financing | filed, subtotal | Net cash provided by (used in) financing activities. |

### Cash reconciliation (`section: cash_reconciliation`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `EffectOfExchangeRateOnCash` | FX effect on cash | filed | Effect of exchange-rate changes on cash, cash equivalents and restricted cash. |
| `NetChangeInCash` | Net change in cash | filed, subtotal | Net increase (decrease) in cash, cash equivalents and restricted cash for the period. |
| `CashAtEndOfPeriod` | Cash at end of period | filed, subtotal | Cash, cash equivalents and restricted cash at period end — the closing line. Ties to BS cash + restricted cash; the prior period's close is this period's opening balance. |

### Free cash flow (`section: free_cash_flow`)
| Concept | Line item | Type | Formula | Description & role |
| :-- | :-- | :-- | :-- | :-- |
| `NetCapEx` | Net capital expenditure | calc | `CapitalExpenditures - DepreciationAmortization` | Cap-ex net of depreciation. Role: part of reinvestment. |
| `ChangeInNonCashWorkingCapital` | Change in non-cash WC | calc | `NonCashWorkingCapital - prior-period NonCashWorkingCapital` | YoY change in non-cash working capital. Role: part of reinvestment. |
| `Reinvestment` | Reinvestment | calc | `NetCapEx + ChangeInNonCashWorkingCapital + Acquisitions` | What the firm plows back. Role: `FCFF = NOPAT - Reinvestment`. |
| `FCFF` | Free cash flow to firm | calc | `NOPAT - Reinvestment` | After-tax operating income less reinvestment. Role: discounted at the cost of capital. |
| `FCFE` | Free cash flow to equity | calc | `NetIncome - NetCapEx - ChangeInNonCashWorkingCapital + (DebtIssued - DebtRepaid)` | Cash flow to equity after reinvestment and net borrowing. Role: discounted at the cost of equity. |

---

## Disclosure (footnote-sourced) — `group="disclosure"`

### Lease disclosure (`section: lease_disclosure`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `OperatingLeaseExpense` | Operating lease expense | filed | Operating-lease / rent expense from the lease footnote. Role: imputed interest add-back when capitalizing leases (esp. pre-ASC842). |
| `OperatingLeaseFutureMinimumPayments` | Future minimum lease payments | filed | Undiscounted future operating-lease commitments (lease footnote schedule). Role: present-valued into capitalized lease debt (pre-ASC842). |

### Equity-compensation disclosure (`section: equity_compensation_disclosure`)
| Concept | Line item | Type | Description & role |
| :-- | :-- | :-- | :-- |
| `EmployeeStockOptionsOutstanding` | Options outstanding | filed | Number of employee stock options outstanding (equity-comp footnote). Role: option-value overhang subtracted from equity value per share. |
| `EmployeeStockOptionsWeightedAvgExercisePrice` | Options WAEP | single | Weighted-average exercise price of options outstanding. Role: strike input to valuing the option overhang. |
| `EmployeeStockOptionsWeightedAvgRemainingTermYears` | Options avg term | single | Weighted-average remaining contractual term of options (years). Role: maturity input to valuing the option overhang. |

---

## Ratios (intrinsic valuation) — `group="ratios"`

All six are calculated. Ratios are dimensionless (formatted to 4 decimals).

### Return ratios (`section: return_ratios`)
| Concept | Line item | Type | Formula | Description & role |
| :-- | :-- | :-- | :-- | :-- |
| `EffectiveTaxRate` | Effective tax rate | ratio | `IncomeTaxExpense / PretaxIncome` | Income taxes over pretax income. |
| `ROIC` | Return on invested capital | ratio | `NOPAT / InvestedCapital` (capital lagged 1 period) | After-tax operating income over invested capital. Role: quality of growth; terminal-value reinvestment. |
| `SalesToCapital` | Sales-to-capital | ratio | `Revenue / InvestedCapital` (capital lagged) | Revenue per unit of invested capital. Role: drives reinvestment in a revenue-growth valuation. |

### Growth ratios (`section: growth_ratios`)
| Concept | Line item | Type | Formula | Description & role |
| :-- | :-- | :-- | :-- | :-- |
| `ReinvestmentRate` | Reinvestment rate | ratio | `Reinvestment / NOPAT` | Reinvestment as a share of after-tax operating income. |
| `ExpectedGrowth` | Expected growth | ratio | `ReinvestmentRate * ROIC` | Fundamental growth. Role: growth in operating income during the high-growth phase. |

### Credit ratios (`section: credit_ratios`)
| Concept | Line item | Type | Formula | Description & role |
| :-- | :-- | :-- | :-- | :-- |
| `InterestCoverage` | Interest coverage | ratio | `OperatingIncome / InterestExpense` | Operating income over interest expense. Role: synthetic credit rating → cost of debt. |
