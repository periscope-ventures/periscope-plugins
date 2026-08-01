# Periscope plugin for Claude Code

A single Claude Code plugin that bundles the **Periscope MCP connector** and Periscope
skills. Install it once and Claude gains the Periscope tools (`fin_*`, `ops_*`,
`watchlist_*`, …) and knows how to reference Periscope data in Excel.

## What's inside

| Component | What it does |
| :-- | :-- |
| **MCP connector** (`.mcp.json`, server key `periscope-mcp`) | Connects to the Periscope gateway at `https://mcp.periscope.ventures/mcp` over streamable HTTP. Auth is OAuth in the browser on first use — no tokens to configure. |
| **`prsc-functions` skill** (`skills/prsc-functions/`) | How to reference any Periscope data element from an Excel cell via the `PRSC.*` custom functions (the Periscope Office add-in) — writing live `=PRSC.FIN.*` / `=PRSC.MACRO.*` formulas that mirror the `fin_*`/`macro_*` MCP tools 1:1. |
| **`standard-financials` skill** (`skills/standard-financials/`) | The standard-concept catalog behind `fin_standard_financials_and_ratios` — every standardized line item across the five groups (income statement, balance sheet, cash flow, disclosure, ratios) with its id, description, formula, and Damodaran valuation role. |

## Install

This repo **is** the plugin (manifest at `.claude-plugin/plugin.json`), distributed
as a single-plugin marketplace.

Add the marketplace, then install:

```bash
claude plugin marketplace add periscope-ventures/periscope-plugins
claude plugin install periscope@periscope-plugins
```

Or, for local development / a quick session without installing:

```bash
claude --plugin-dir /path/to/periscope-plugins
```

On first use of a Periscope tool, Claude Code opens the OAuth flow in your browser.

### Claude Desktop

Add the marketplace and install the plugin the same way from the Claude Code CLI;
the plugin is then available in your synced Claude environment.

## Layout

```
periscope-plugins/                     <- repo root = the marketplace
├── .claude-plugin/
│   └── marketplace.json               <- lists the plugins below
└── plugins/
    └── periscope/                     <- the plugin
        ├── .claude-plugin/
        │   └── plugin.json            <- manifest
        ├── .mcp.json                  <- Periscope HTTP MCP connector (server key: periscope-mcp)
        ├── skills/
        │   ├── prsc-functions/        <- using the PRSC.* Excel custom functions
        │   │   ├── SKILL.md
        │   │   └── references/
        │   │       └── functions.md   <- full per-function parameter reference
        │   └── standard-financials/   <- the standard-concept catalog (reports & ratios)
        │       ├── SKILL.md
        │       └── references/
        │           └── concepts.md    <- all 111 concepts, grouped by statement & section
        └── README.md
```

## Develop

Validate the manifest and components before publishing:

```bash
claude plugin validate . --strict
```

Because `version` is set in `plugin.json`, bump it whenever you want installed users
to receive changes — pushing commits alone won't trigger an update.
