# Periscope plugin for Claude Code

A single Claude Code plugin that bundles the **Periscope MCP connector**. Install it
once and Claude gains the Periscope tools (`fin_*`, `ops_*`, `watchlist_*`, …).

## What's inside

| Component | What it does |
| :-- | :-- |
| **MCP connector** (`.mcp.json`, server key `periscope-mcp`) | Connects to the Periscope gateway at `https://mcp.periscope.ventures/mcp` over streamable HTTP. Auth is OAuth in the browser on first use — no tokens to configure. |

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
        └── README.md
```

## Develop

Validate the manifest and components before publishing:

```bash
claude plugin validate . --strict
```

Because `version` is set in `plugin.json`, bump it whenever you want installed users
to receive changes — pushing commits alone won't trigger an update.
