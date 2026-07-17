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
| `repoInfo` | `{repo: "purpose text"}` — the `!` explanations in each view's repo table (falls back to the repo's GitLab description) |
| `services` | platform-run services drawn on the map: `[{name, kind, layer, stage, regions, repo, icon, info}]`. A `kind:"gateway"` service may declare a flow — `from`/`fromVia` (external consumer + link label) and `to`/`toVia` (backend service names) — drawn as one continuous line on the globe's AI layer and in the region view. A backend with `geo:{lat,lng,label}` is placed at its real location (cross-region hops stay honest) |
| `helmChartsRepo` | repo whose `**/Chart.yaml` files feed the live chart list (name·version·description) shown in the Helm Charts view — re-read every sync |
| `moduleReleasesWiki` | group-wiki page slug holding the module release table (`\| name \| version \| date \|`) — parsed every sync into the Terraform Modules view (files source: `<platformDir>/.wiki/<slug>.md`) |
| `ciComponentsRepo` | repo whose `templates/*.yml` files (the GitLab CI/CD component convention, incl. `templates/<name>/template.yml`) feed the live CI component catalog — name, `spec.inputs` names and the leading comment as description, re-read every sync |
| `backups` | backup posture drawn as its own map layer: `{repo, stage, regions:[codes], vaults, enrolment, info, geo:[…]}`. Each `geo` entry is a **cross-region story** — `{from, lat, lng, label}` (geo-replica to a point, e.g. the cloud's paired region) or `{from, toRegion, label}` (a DR pairing between two platform regions). The **globe draws only these** (vault markers appear only on regions an arc touches); every covered region still gets its vault chip in the region view |

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
