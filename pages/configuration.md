# Configuration

One JSON file per deployment (default `server/config/platform.config.json`, override with
`--config` / env `CONFIG`).

| Key | Meaning |
| --- | --- |
| `group` | the platform group path, e.g. `my-org/platform` |
| `gitlabUrl` | instance base URL (default `https://gitlab.com`) |
| `platformDir` | **files source**: directory describing the platform — its presence selects the source; omit it for the gitlab source |
| `sectors` | sector names for environment parsing (`<sector>-<stage>-<region>`) |
| `sectorAliases` | `{ "short": "canonical" }` env-name aliases |
| `viewMap` | the map's views → repos: `[{view, repos:[…]}]` or `{view, prefix:"path/"}` — repos by **group-relative path** (`team/repo`; bare slug accepted for top-level repos) |
| `appSources` | where applications live: `"tenantsRepo:dir,namespacesRepo:dir"` |
| `networkRepo` | the repo whose `**/subnet.<name>.tf` files feed the subnet inventory |
| `tenantPermissions` | `{default, extended}` repos — powers the tenant-permissions chat answer |
| `knowledge` | curated facts injected into every chat answer |
| `sourceLabel` / `sourceNote` | UI header + badge text for this deployment |
| `drift` | optional externally-computed cloud-vs-code drift feed for the drift panel |

## Flags & environment

| | |
| --- | --- |
| `--group · --interval · --port · --platform-dir · --config` | flags override env, env overrides config |
| `GITLAB_TOKEN` | `read_api` PAT — required for the gitlab source, never logged |
| `OVERLAY_PATH` | view-map overlay location (reconciliation state) |
| `STATE_CACHE` | last-good `/live` snapshot for instant restarts |
| `SECURITY_SCAN_INTERVAL` | terraform security scan period (s, default 600) |
| `ANTHROPIC_API_KEY` / `AUTO_RECONCILE=1` | grounded chat / LLM drift reconciliation |
| `UI_DIR` / `DOCS_DIR` | where the UI and the markdown notes are served from |

## The overlay

`view-map.local.json` extends the config's `viewMap` without touching it:
`assignments` (view → extra repos), `newViews`, `ignore` (repos/prefixes the drift check
skips), `removeExpected`, `excludeApps`. It is written by the reconciler and re-read every
sync — suggestions apply live.
