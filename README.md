# whats-new.cloud — Kiro Power

A Kiro Power that verifies recent **AWS, Google Cloud, Microsoft Azure, and Oracle
Cloud (OCI)** feature launches (What's New / release notes) against each vendor's
**original English source text**. Useful when currency-reviewing cloud documentation:
confirm GA/Preview status, launch dates, regions, and feature details from the
authoritative source rather than a localized summary.

It connects to the `cloud-whats-new` MCP server at `https://whats-new.cloud/mcp`,
which ingests the CSPs' official "What's New" / release-notes feeds.

## What it provides

An MCP server (`cloud-whats-new`) with three read-only tools:

| Tool | Purpose |
| --- | --- |
| `search_releases` | Search release notes by keyword / CSP / date range |
| `get_release` | Fetch one release note by article ID |
| `get_stats` | Translation/review pipeline status |

Coverage:

- **CSPs:** AWS, Google Cloud, Azure, Oracle Cloud (OCI) — `csp` lowercase.
- **Languages:** Korean (`ko`, default), English (`en`), Japanese (`ja`).

Two output formats:

- `format:"summary"` (default, **no auth**) — human-readable per-language summary.
- `format:"source"` (**requires a Bearer token**) — the vendor-original English text
  plus the official `url`. Use this for source-of-truth verification.

## Installation

Install from this GitHub repository via the Kiro Powers UI (Add Custom Power → import
from Git/folder). Once installed it is available in both the Kiro IDE and CLI, and
activates dynamically based on the keywords in `POWER.md` (e.g. "aws-whats-new",
"oci-updates", "release-notes", "ga-preview-status").

## Authentication (token setup)

`format:"source"` calls require a Bearer token. **The token is never stored in this
repository** — the bundled `mcp.json` only references an environment variable:

```json
"headers": { "Authorization": "Bearer ${WNK_MCP_TOKEN}" }
```

You supply the value yourself. The recommended setup keeps the token off disk in
plaintext:

1. Store the token in the macOS Keychain:
   ```bash
   security add-generic-password -a "$USER" -s wnk-mcp-token -w "wnk_mcp_..." -U
   ```
2. Export it from the Keychain in your shell startup (`~/.zshenv`), so every Kiro
   process (IDE or CLI) gets it:
   ```bash
   export WNK_MCP_TOKEN="$(security find-generic-password -w -s wnk-mcp-token 2>/dev/null)"
   ```

The token itself lives only in (a) the server-side Cloudflare Worker secret
(`API_KEY_RING`, type `mcp`) that the endpoint checks against, and (b) your local
Keychain. It is not committed here, not written into `mcp.json`, and not exported as a
literal in any dotfile.

Without a valid token, `format:"summary"`, `get_stats`, and tool discovery still work;
only `format:"source"` returns `-32001` until the token is present.

## Files

| File | Purpose |
| --- | --- |
| `POWER.md` | Power manifest (frontmatter + usage guidance for the agent) |
| `mcp.json` | MCP server config (references `${WNK_MCP_TOKEN}`, no secret) |
| `LICENSE` | MIT-0 |

## Endpoint note

The canonical endpoint is `https://whats-new.cloud/mcp`. The legacy
`https://api.whats-new.kr/mcp` URL still works but 301-redirects to the canonical host.

## Scope

Covers the major CSPs (AWS / GCP / Azure / OCI) only. Model vendors (OpenAI /
Anthropic / Google 1P), regulators, and domestic FM providers are out of scope and
stay on manual web cross-checking. Read-only; no write/admin tools.
