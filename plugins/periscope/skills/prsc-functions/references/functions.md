# `PRSC.*` function reference

Complete parameter tables, return shapes, and examples for the six Periscope Excel custom
functions. Arguments are **positional** — the order below is the order you type them in the
cell. `identifier` is a ticker or CIK (`"AAPL"` or `"320193"`). The two FIN report
functions default to the last **3** `10-K` filings.

Each function is a 1:1 wrapper over a Periscope MCP tool (noted per function as **MCP
twin**) with the same parameters — call the tool to discover/validate arguments, then emit
the matching `=PRSC.*` formula.

---

## `PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS`

MCP twin: `fin_standard_financials_and_ratios`.

A company's **standardized** financials & ratios, merged over the last N filings of a form
— the five groups (income statement / balance sheet / cash flow / disclosure / ratios) as a
concept-rows × period-columns grid. Tags are normalized onto a common concept catalog so
periods and companies line up.

| # | Parameter | Type | Required | Description |
| :-- | :-- | :-- | :-- | :-- |
| 1 | `identifier` | string | yes | Company CIK or ticker, e.g. `"AAPL"` or `"320193"`. |
| 2 | `form` | string | no | Filings to span, e.g. `"10-K"` (default) or `"10-Q"`. Mutually exclusive with `accessions`. |
| 3 | `last_n_filings` | number | no | How many filings of this form to merge, most recent first (default 3, max 20). |
| 4 | `accessions` | range/list | no | Merge these exact accession numbers instead. Mutually exclusive with `form`. |
| 5 | `entity` | string | no | Reporting entity (a `dei:LegalEntityAxis` member) for a combined filer; omit for the consolidated parent. |
| 6 | `group` | string | no | Narrow to one group: `"income_statement"`, `"balance_sheet"`, `"cash_flow"`, `"disclosure"`, or `"ratios"`. |

**Returns** — a matrix: row 1 is the title / a "Formula / notes" column / one column per
period (earliest → latest); rows below are concept rows (indented by section), each with its
label, notes, and per-period value.

**Examples**
- `=PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS("AAPL")` — last 3 10-Ks, all groups.
- `=PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS("MSFT","10-Q",8)` — last 8 quarters.
- `=PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS("AAPL",,,,,"ratios")` — just the ratios group
  (note the five empty commas to skip `form`, `last_n_filings`, `accessions`, `entity`).

---

## `PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME`

MCP twin: `fin_company_financials_over_time`.

A company's financials **as filed**, over time — every statement/disclosure across N
filings, merged. Progressive: `view="index"` lists the sections; `view="detail"` returns a
records table, one row per as-filed fact.

| # | Parameter | Type | Required | Description |
| :-- | :-- | :-- | :-- | :-- |
| 1 | `identifier` | string | yes | Company CIK or ticker, e.g. `"AAPL"` or `"320193"`. |
| 2 | `form` | string | no | Filings to span, e.g. `"10-K"` (default) or `"10-Q"`. Mutually exclusive with `accessions`. |
| 3 | `last_n_filings` | number | no | How many filings of this form to merge, most recent first (default 3, max 20). |
| 4 | `accessions` | range/list | no | Merge these exact accession numbers instead. Mutually exclusive with `form`. |
| 5 | `view` | string | no | `"index"` (default, the section list) or `"detail"` (a records table of the selected sections' facts). |
| 6 | `category` | string | no | Narrow to one menu group: `"Statements"`, `"Details"`, `"Cover"`, `"Notes"`, `"Tables"`, or `"Policies"`. |
| 7 | `section` | string | no | Case-insensitive substring matched against a section's title or role, e.g. `"balance"`. |
| 8 | `undimensioned` | boolean | no | `TRUE` to drop dimensional (per-axis) rows, keeping only the consolidated totals. |
| 9 | `reveal_mapping` | boolean | no | `TRUE` to add a "Standard concept" column — the catalog concept each row feeds in the standardized reports/ratios. |

**Returns**
- `view="index"`: a section index — Category, Statement type, Section, Lines, Role.
- `view="detail"`: a records table — Section, Label, Concept, [Standard concept (if
  `reveal_mapping`)], Dimensions, Period key, Period end, Value, Unit, Accession, Provenance.

**Examples**
- `=PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME("AAPL")` — the section index for the last 3 10-Ks.
- `=PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME("AAPL","10-K",3,,"detail",,"balance")` — the
  as-filed balance-sheet facts.
- `=PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME("AAPL","10-K",3,,"detail",,"balance",TRUE)` — same,
  consolidated totals only (drop dimensional rows).

Typical flow: run it with defaults to browse the index, find the section title you want, then
re-run with `view="detail"` and a `section=` (or `category=`) narrow.

---

## `PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME_ITEMS`

MCP twin: `fin_company_financials_over_time_items`.

Filter the over-time as-filed records to the line(s) you name and return their **value(s)** —
a numeric spill you can `SUM` or drop into a cell. Same filing window + dedupe rules as
`COMPANY_FINANCIALS_OVER_TIME`, just filtered.

| # | Parameter | Type | Required | Description |
| :-- | :-- | :-- | :-- | :-- |
| 1 | `identifier` | string | yes | Company CIK or ticker, e.g. `"META"` or `"320193"`. |
| 2 | `form` | string | no | Filings to span, e.g. `"10-K"` (default) or `"10-Q"`. |
| 3 | `last_n_filings` | number | no | How many filings to merge, most recent first (default 3). |
| 4 | `section` | string | no | Only read sections whose title/role contains this, e.g. `"Maturities of Long-Term Debt"`. |
| 5 | `standard_concept` | string | no | Keep rows mapped to this catalog concept (exact), e.g. `"revenue"`. |
| 6 | `concept` | string | no | Keep rows whose raw XBRL concept contains this, e.g. `"LongTermDebtMaturities"`. |
| 7 | `label` | string | no | Keep rows whose filed label contains this, e.g. `"2027"` (a schedule's future bucket). |
| 8 | `period` | string | no | Keep rows in this period — a year `"2024"`, an ISO date, or a period key. **Not** for maturity buckets (use `label`). |
| 9 | `dimensions` | string | no | Keep rows whose axis=member signature contains this, e.g. `"GasMember"`. |
| 10 | `unit` | string | no | Keep rows with this exact unit — `"usd"` / `"shares"` / `"usdPerShare"`. |

**Returns** — a numeric spill (one column, one row per matching fact), formatted with the
right unit/decimals and ready to `SUM`. Returns a message when nothing matches.

**Examples**
- `=PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME_ITEMS("AAPL",,,,"revenue")` — the revenue value(s).
- `=SUM(PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME_ITEMS("META","10-K",1,"Maturities of Long-Term Debt",,,"2027"))`
  — Meta's long-term debt maturing in 2027 (bucket lives in `label`, note the empty `period`).

> **The maturity-bucket trap:** a future-year debt/lease payment is `label="2027"`, not
> `period="2027"`. `period` filters real fiscal periods; schedule/future buckets are labels.

---

## `PRSC.MACRO.GET_RISK_FREE_RATE`

MCP twin: `macro_get_risk_free_rate`.

The clean risk-free rate for a currency — the anchor of a cost of capital — as a labeled
block.

| # | Parameter | Type | Required | Description |
| :-- | :-- | :-- | :-- | :-- |
| 1 | `currency` | string | no | Currency the valuation is denominated in, e.g. `"USD"` (default). |
| 2 | `real` | boolean | no | `FALSE` (default) for a nominal rate; `TRUE` for a real (inflation-indexed) rate. |

**Returns** — a 2-row labeled block: `currency`, `real`, `rate`, `rate_percent`,
`source_series` (the FRED series used), `date` (the observation date).

**Examples**
- `=PRSC.MACRO.GET_RISK_FREE_RATE()` — nominal USD.
- `=PRSC.MACRO.GET_RISK_FREE_RATE("USD",TRUE)` — real (inflation-indexed) USD.

---

## `PRSC.MACRO.GET_RATES`

MCP twin: `macro_get_rates`.

Latest value for each requested rate series — Treasury constant-maturity yields by tenor
(FRED `DGS3MO` … `DGS30`).

| # | Parameter | Type | Required | Description |
| :-- | :-- | :-- | :-- | :-- |
| 1 | `series` | range/list | yes | One or more FRED rate series ids, e.g. `"DGS10"`, `"DGS2"`. Pass a range (`A1:A3`) or array constant (`={"DGS2","DGS10"}`). |

Common tenors: `DGS3MO`, `DGS6MO`, `DGS1`, `DGS2`, `DGS3`, `DGS5`, `DGS7`, `DGS10`, `DGS20`,
`DGS30`.

**Returns** — a matrix, one row per requested series: `series_id`, `date`, `value`.

**Examples**
- `=PRSC.MACRO.GET_RATES("DGS10")` — latest 10-year Treasury yield.
- `=PRSC.MACRO.GET_RATES({"DGS2","DGS10","DGS30"})` — a mini yield-curve snapshot.

---

## `PRSC.MACRO.GET_RATE_SERIES`

MCP twin: `macro_get_rate_series`.

Time series for each requested rate series over `[start, end]` — Treasury yields by tenor.

| # | Parameter | Type | Required | Description |
| :-- | :-- | :-- | :-- | :-- |
| 1 | `series` | range/list | yes | One or more FRED rate series ids, e.g. `"DGS10"`, `"DGS2"`. |
| 2 | `start` | string | no | ISO `YYYY-MM-DD` lower bound (inclusive); omit for full history. |
| 3 | `end` | string | no | ISO `YYYY-MM-DD` upper bound (inclusive); omit for up-to-latest. |

**Returns** — a long-format matrix, one row per observation: `series_id`, `date`, `value`.
Multiple series stack into one rectangular block.

**Examples**
- `=PRSC.MACRO.GET_RATE_SERIES("DGS10","2020-01-01")` — 10-year yield from 2020 to latest.
- `=PRSC.MACRO.GET_RATE_SERIES({"DGS2","DGS10"},"2020-01-01","2024-12-31")` — two tenors over
  a fixed window.
