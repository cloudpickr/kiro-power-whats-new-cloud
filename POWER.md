---
name: "cloud-whats-new"
displayName: "Cloud What's New"
description: "Verify recent AWS, Google Cloud, and Microsoft Azure feature launches (What's New / release notes) against the vendors' original English source text. Power for currency-checking cloud docs; source-format access is token-gated."
keywords: ["cloud-whats-new", "aws-whats-new", "gcp-release-notes", "azure-updates", "ga-preview-status", "release-notes"]
author: "froguin"
---

# Cloud What's New

## Overview

Power that connects to the **cloud-whats-new** MCP server (`https://api.whats-new.kr/mcp`) to look up recent release notes from the three major cloud providers — AWS, Google Cloud, Microsoft Azure. The server ingests each vendor's official "What's New" / release-notes RSS feed.

Scope for our use: **verifying the three CSPs' recent feature launches** (GA/Preview status, launch dates, regions, features) when currency-reviewing docs. It is **not** a general multi-vendor SOT — model vendors (OpenAI/Anthropic/Google 1P), regulators, and domestic FM providers stay on manual web cross-checking. This power complements the AWS Knowledge MCP (AWS-only, deeper) by covering all three CSPs.

This power is installed from its public GitHub repository (`kiro-power-whats-new-kr`). The auth token itself is **never** stored in the repo — only a `${WNK_MCP_TOKEN}` reference is (see MCP Config Placeholders below).

## Source-of-truth usage (important)

For updating docs, we need the **vendor original English text, not the localized summary.** The MCP therefore exposes a hidden `format` parameter (not shown in the public `tools/list` schema):

- `format:"summary"` (default, no auth) — human-readable per-language summary. Not for SOT.
- `format:"source"` (**requires `Authorization: Bearer <wnk_mcp_...>`**) — returns the vendor-original English text + official `url`, independent of translation status. **Use this for SOT verification.** The `url` is the final source of truth; the MCP finding is the first-pass index.

Because `format:"source"` is intentionally omitted from the public schema, pass it explicitly in the tool arguments.

## Available MCP Server

**Server:** `cloud-whats-new` (remote HTTP JSON-RPC, `https://api.whats-new.kr/mcp`).
**Auth:** `Authorization: Bearer ${WNK_MCP_TOKEN}` — required for `format:"source"` calls.

### Tools

| Tool | Purpose | Key parameters |
| --- | --- | --- |
| `search_releases` | Search release notes by keyword / CSP / date range | `query` (string), `csp` (`aws`\|`gcp`\|`azure`), `lang` (`ko`\|`en`), `days`, `start_date`/`end_date` (`YYYY-MM-DD`), `limit`, `format` (`summary`\|`source`) |
| `get_release` | Fetch one release note by article ID | `id` (number, required), `format` (`summary`\|`source`) |
| `get_stats` | Translation/review pipeline status | none |

### Semantics

- `csp` must be lowercase (`aws`, `gcp`, `azure`).
- With `format:"source"`, results always include the vendor-original text and `url`, regardless of translation backlog.
- `query` keyword search matches the summary text; when a keyword may not match, omit `query` and filter by `csp` + date window, then read results.

## Tool Usage Examples

Verify an AWS feature's source text and date (SOT — token required):

```
search_releases({ "csp": "aws", "format": "source", "start_date": "2026-09-01", "end_date": "2026-09-12", "limit": 20 })
```

Read one release's original source + official URL by ID:

```
get_release({ "id": 4336634, "format": "source" })
```

## Best Practices

- Use `format:"source"` for anything that will change a doc; cite the returned `url` as the authoritative source and confirm the final fact there.
- Scope to the three CSPs only. For model-vendor / regulatory / domestic-FM facts, use manual web cross-checking, not this power.
- Prefer `csp` + date-window scans over keyword search when phrasing is uncertain.
- Read-only reference; no write/admin tools.

## MCP Config Placeholders

- **`WNK_MCP_TOKEN`**: Bearer token (`wnk_mcp_...`) authenticating `format:"source"` calls to `https://api.whats-new.kr/mcp`.
  - The token is an `API_KEY_RING` entry of `"type": "mcp"` on the deployed `cloud-whats-new` Worker. Manage/issue it yourself; never write the literal token into `mcp.json` or a shared session.
  - Provide it as an environment variable in the shell that launches Kiro:
    ```bash
    export WNK_MCP_TOKEN="wnk_mcp_...."
    ```
  - Confirm this Kiro build expands `${WNK_MCP_TOKEN}` inside `mcp.json` `headers`; if not, supply the token via the client's supported env mechanism.

## Troubleshooting

### `-32001` "format=\"source\" requires Authorization: Bearer <token>"
`format:"source"` was called without a valid token. Ensure `WNK_MCP_TOKEN` is set and matches an `API_KEY_RING` `mcp` entry on the deployed Worker.

### Summary returned instead of source
No `format:"source"` was passed (default is `summary`). Pass it explicitly — it is not in the public schema.

### Connection errors
Confirm the endpoint is `https://api.whats-new.kr/mcp` (the Worker is on the `api.` subdomain; `whats-new.kr` is the website host).

---

**MCP Server:** cloud-whats-new (`https://api.whats-new.kr/mcp`)
**Auth:** `Authorization: Bearer ${WNK_MCP_TOKEN}` (required for `format:"source"`)
