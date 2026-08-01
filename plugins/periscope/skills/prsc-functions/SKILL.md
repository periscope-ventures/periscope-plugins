---
name: prsc-functions
description: How to reference any Periscope data element from an Excel cell using the PRSC.* custom functions (the Periscope Office add-in). Use this skill whenever you need to put Periscope data into a spreadsheet — a company's standardized financials or ratios, any as-filed statement/footnote fact, a specific line item, or a Treasury/risk-free rate — by writing a live =PRSC.* formula instead of pasting a static number, or when a user asks how to write/read/fix such a formula, what a PRSC.FIN.* or PRSC.MACRO.* function takes, or why it spills or errors. The PRSC functions mirror the Periscope MCP fin_*/macro_* tools 1:1, so you can discover the exact identifier/concept/section/accession with the MCP tool and then emit the matching =PRSC.* formula. Triggers: "put AAPL's financials in Excel", "reference Meta's 2027 debt maturities in a cell", "what's the Excel formula for the 10-year Treasury", "PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS parameters", "the Periscope Excel function is erroring".
---

# Referencing Periscope data in Excel (`PRSC.*`)

The Periscope Office add-in adds six **custom functions** to Excel under the `PRSC`
namespace. They pull live data from the Periscope platform — standardized financials,
any as-filed XBRL fact, single line items, and Treasury/risk-free rates — and **spill**
the result into the sheet.

This skill is for an agent that needs to land a Periscope data element in a spreadsheet.
**Prefer writing a live `=PRSC.*` formula over pasting a static number** — the cell then
stays tied to the source, recalculates, and shows provenance. The point is to be able to
reference *any* data element, so the core skill is turning "I need figure X for company Y"
into the exact formula that fetches it.

Full per-parameter tables and more examples are in
[references/functions.md](references/functions.md) — read it before emitting or debugging
any specific formula; arguments are positional, so exact order and allowed values matter.

## What you can reference

- **Any standardized concept or ratio** → `PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS`.
  Tags normalized onto a common catalog (income statement / balance sheet / cash flow /
  disclosure / ratios), so periods and companies line up.
- **Any as-filed fact** — any statement, footnote, schedule, or dimensional breakout the
  company actually filed → `PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME` (explore) and
  `PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME_ITEMS` (pull the value).
- **Rates** — the risk-free rate for a currency, and Treasury constant-maturity yields
  (latest or a time series) → the three `PRSC.MACRO.*` functions.

Between them, the FIN functions can reference essentially any financial data element (raw
or standardized) and MACRO covers rates. The live add-in surface is whatever autocomplete
shows after you type `=PRSC.` (and the add-in manifest is the source of truth) — today
that is the six functions below. The same patterns apply if more `PRSC.*` categories ship.

## Discover with MCP, emit as a formula

Each `PRSC.*` function is a 1:1 wrapper over a Periscope MCP tool — **same parameters,
same behavior**:

| Excel function | Periscope MCP tool |
| :-- | :-- |
| `PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS` | `fin_standard_financials_and_ratios` |
| `PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME` | `fin_company_financials_over_time` |
| `PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME_ITEMS` | `fin_company_financials_over_time_items` |
| `PRSC.MACRO.GET_RISK_FREE_RATE` | `macro_get_risk_free_rate` |
| `PRSC.MACRO.GET_RATES` | `macro_get_rates` |
| `PRSC.MACRO.GET_RATE_SERIES` | `macro_get_rate_series` |

So when you don't already know the exact ticker/CIK, standard concept, section title, or
accession, **call the MCP tool first to discover and validate it, then translate the same
arguments into the `=PRSC.*` formula** you write into the cell. (`fin_lookup_company` /
`fin_search_companies` resolve an identifier; running `fin_company_financials_over_time`
with `view="index"` reveals section titles; `reveal_mapping`/`fin_reconcile` confirm the
standard concept a raw row feeds.) The MCP call proves the formula will return what you
intend before it ever lands in the workbook.

## How the functions are called in Excel

- **Namespace + category + name:** every function is `=PRSC.<CATEGORY>.<NAME>(...)`, e.g.
  `=PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS("AAPL")`. Autocomplete lists them after `=PRSC.`.
- **Results spill.** Each returns a 2-D block that spills down and right from its cell.
  Leave room, or the cell shows `#SPILL!`. Reference the spill elsewhere with `#`, e.g.
  `=SUM(A1#)`.
- **Arguments are positional** — no named arguments in a cell. To skip an optional
  argument in the middle, leave it empty between commas, e.g. the ratios group only (6th
  arg): `=PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS("AAPL",,,,,"ratios")`.
- **List/range arguments** (`accessions`; `series` in MACRO) take a cell range (`A1:A3`)
  or an array constant (`={"DGS2","DGS10"}`).
- **Values stay numeric.** Figures spill as real numbers with display formatting (`M`/`K`/
  `B` suffixes are cosmetic), so `SUM`, charts, and downstream formulas work.
- **Not volatile** — they recalc on input/formula change or a forced recalc, not on every
  edit.
- **Auth:** the functions authenticate through the add-in task pane. An auth-type `#VALUE!`
  means the task pane isn't signed in — open it (**Home → Add-ins**, or the Periscope
  ribbon button) and sign in with Periscope (Cognito) credentials; the token is shared
  with the functions runtime. The add-in targets Excel on the web (and desktop) and is
  deployed via the Microsoft 365 admin center.

## The six functions

| Function | Reference it to get… | Minimal example |
| :-- | :-- | :-- |
| `PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS` | A standardized statements + ratios grid (concept rows × period columns). | `=PRSC.FIN.STANDARD_FINANCIALS_AND_RATIOS("AAPL")` |
| `PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME` | As-filed statements across filings — `view="index"` lists sections, `view="detail"` returns facts. | `=PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME("AAPL")` |
| `PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME_ITEMS` | A specific line-item value (or SUM-ready column), filtered by concept/label/section/period. | `=PRSC.FIN.COMPANY_FINANCIALS_OVER_TIME_ITEMS("AAPL",,,,"revenue")` |
| `PRSC.MACRO.GET_RISK_FREE_RATE` | The risk-free rate for a currency. | `=PRSC.MACRO.GET_RISK_FREE_RATE("USD")` |
| `PRSC.MACRO.GET_RATES` | The latest value for one or more Treasury tenors (FRED `DGS*`). | `=PRSC.MACRO.GET_RATES("DGS10")` |
| `PRSC.MACRO.GET_RATE_SERIES` | A time series for Treasury tenors over a date range. | `=PRSC.MACRO.GET_RATE_SERIES("DGS10","2020-01-01")` |

`identifier` is a ticker or CIK (`"AAPL"` or `"320193"`). The two FIN report functions
default to the last **3** `10-K` filings; pass `form="10-Q"` and/or `last_n_filings` to
change the window.

## Choosing the right FIN function for a data element

- **A normal, comparable statement value or a ratio** → `STANDARD_FINANCIALS_AND_RATIOS`,
  narrowing with `group=` (`income_statement`, `balance_sheet`, `cash_flow`, `disclosure`,
  `ratios`). Best when the element maps to a common concept and you want periods aligned.
- **Exactly what the company filed** (footnotes, schedules, dimensional/per-segment
  breakouts) → `COMPANY_FINANCIALS_OVER_TIME` with `view="index"` to locate the section,
  then `view="detail"` + a `section=`/`category=` narrow to see the facts.
- **One number, or a column to SUM** → `COMPANY_FINANCIALS_OVER_TIME_ITEMS`: the filtered,
  values-only spill. This is the workhorse for wiring a single element into a model cell.

## Workflow: land an arbitrary data element in a cell

1. **Resolve the company.** Ticker or CIK; if unsure, `fin_lookup_company` /
   `fin_search_companies` (MCP).
2. **Locate the element.** Known standard concept/ratio → STANDARD. Otherwise browse with
   `COMPANY_FINANCIALS_OVER_TIME` `view="index"` (MCP or formula) to find the section, and
   use `reveal_mapping=TRUE` to see which standard concept a raw row feeds.
3. **Validate with the MCP twin.** Call the matching `fin_*`/`macro_*` tool with the args
   you intend, confirm it returns the element you want (right value, unit, period).
4. **Emit the formula.** Translate those exact args (in positional order — see
   [references/functions.md](references/functions.md)) into the `=PRSC.*` call, place it
   where it has room to spill, and reference the target cell.
5. **Note the gotchas** that apply (below).

## Gotchas to get right

- **Maturity/schedule buckets live in `label`, not `period`.** A future-year debt or lease
  payment — "long-term debt due in 2027" — is `label="2027"` (plus a `section=`/`concept=`
  narrow), **not** `period="2027"`. Use `period` only for real fiscal-year figures. This is
  the most common mistake with `COMPANY_FINANCIALS_OVER_TIME_ITEMS`.
- **`form` and `accessions` are mutually exclusive** — span recent filings of a type *or*
  name exact accessions, never both.
- **Standardized rows depend on a filed mapping.** If `STANDARD_FINANCIALS_AND_RATIOS` is
  sparse for an obscure filer, that filing may not be mapped yet — not a formula error;
  fall back to `COMPANY_FINANCIALS_OVER_TIME` (as-filed) for the raw facts.
- **`#SPILL!`** = the output range is blocked; clear cells below/right or move the formula.
  **`#VALUE!`** carries the tool's message (bad ticker, not signed in, both `form` and
  `accessions`, etc.) — read it.
- **Fiscal-period labels can be offset** from the calendar year for some filers; if an FY
  column looks off, cross-check the discrete quarters.
