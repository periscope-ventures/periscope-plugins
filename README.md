# Periscope plugin for Claude Code

A single Claude Code plugin that bundles the **Periscope MCP connector** and the
**financial data-quality audit skill**. Install it once and Claude gains both the
Periscope tools (`fin_*`, `ops_*`, `watchlist_*`, …) and the `/audit` workflow that
drives them.

## What's inside

| Component | What it does |
| :-- | :-- |
| **MCP connector** (`.mcp.json`) | Connects to the Periscope gateway at `https://mcp.periscope.ventures/mcp` over streamable HTTP. Auth is OAuth in the browser on first use — no tokens to configure. |
| **`audit` skill** (`skills/audit/`) | Audit and repair a company's financial facts: diagnose, apply the narrowest safe fix, re-verify against SEC filings, and log what can't be auto-fixed. Auto-invokes on requests like "audit AMZN" or "why is debt-to-equity broken for X". |

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
periscope-plugins/                 <- repo root = the plugin
├── .claude-plugin/
│   ├── plugin.json                <- manifest
│   └── marketplace.json           <- lists this plugin so `marketplace add` works
├── .mcp.json                      <- Periscope HTTP MCP connector (server key: gateway)
├── skills/
│   └── audit/
│       ├── SKILL.md
│       └── references/
│           ├── checks.md          <- what each audit check means
│           └── patterns.md        <- concrete fix pattern per problem
└── README.md
```

## Develop

Validate the manifest and components before publishing:

```bash
claude plugin validate . --strict
```

Because `version` is set in `plugin.json`, bump it whenever you want installed users
to receive changes — pushing commits alone won't trigger an update.
