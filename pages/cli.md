# The CLI

`server/cli.mjs` is the terminal companion to the live server — the same `/live` state the
3D map renders, formatted for a tmux pane. It is read-mostly: the only thing it ever writes
is the view-map **overlay** (never the config, never the platform).

## Invocations

```bash
node cli.mjs                                   # full-screen watch (refresh 5s)
node cli.mjs --server http://localhost:8788    # point at any deployment
node cli.mjs --once                            # one render, then exit — CI/script friendly
node cli.mjs --refresh 10                      # watch cadence (the SERVER still syncs on its own interval)
node cli.mjs --skill                           # hand the current drift to Claude (see below)
node cli.mjs --suggest [--exclude a/,b] [--dry]# one structured LLM reconciliation proposal
```

| Flag | Meaning |
| --- | --- |
| `--server <url>` | which deployment to read (default `http://localhost:8787`) |
| `--once` | single render + exit — useful in scripts and screenshots |
| `--refresh <s>` | watch-mode redraw period (default 5s; data freshness is bounded by the server's sync interval) |
| `--skill` | launch `claude` with the **improve-platform-map** skill and a summary of the current drift as the prompt |
| `--suggest` | one schema-validated LLM call proposing view assignments for unmapped repos |
| `--exclude p/,repo` | prefixes/repos `--suggest` must leave alone |
| `--dry` | print the `--suggest` proposal without writing it |
| `OVERLAY_PATH` (env) | where `--suggest` writes its overlay (default `view-map.local.json` next to the CLI) |

## What the screen shows

Top to bottom, all live from `GET /live`:

- **Views → repos** — every view from the deployment's `viewMap`, each expected repo with a
  ✓/✗ found flag and its detected features (`terraform`, `helm`, `ci`, …). This is the
  mapping the 3D map draws, in text.
- **Subscriptions** — the sector × stage rollup parsed from environment names
  (`<sector>-<stage>-<region>`), with per-stage region codes.
- **Helm charts** — every chart (`name@version`) found in the configured chart-library repo
  (`helmChartsRepo`), re-read on each sync.
- **Terraform module releases** — the released module versions parsed from the group wiki's
  release table (`moduleReleasesWiki`), re-read on each sync.
- **CI components** — every CI/CD component template in the configured components repo
  (`ciComponentsRepo`), with its input names and description, re-read on each sync.
- **Terraform security inventory** — every `azurerm_*` resource that grants permissions or
  shapes the network (role assignments, custom roles, subnets, firewall rules, NSG rules,
  peerings, private endpoints), each as `{repo, file, names[]}`. Rescanned every
  `SECURITY_SCAN_INTERVAL` (default 600s).
- **Deploy variables** — per repo, the CI/CD variable **names** (never values) that are
  *defined* on the project vs *referenced* by its `.gitlab-ci.yml`. A referenced-but-undefined
  name is how a broken deploy announces itself before it runs.
- **The drift footer** — the CLI's most important line:

```
⚠ drift: 2 unmapped repos · 0 expected-but-absent — run node cli.mjs --skill to reconcile with the visualization
```

## `--skill` — drift handled by an agent

`--skill` fetches `/live`, summarizes the drift (unmapped repos, absent-but-expected repos,
live regions the map doesn't draw), and launches `claude` with the
**improve-platform-map** skill and that summary as the prompt, cwd'd to the deployment. The
skill's *live drift workflow* then does what a human would: assign repos to views (or the
ignore list), extend the visualization when the drift is structural, mirror the change into
the showcase's fake platform, and re-run the CLI to confirm the footer is clean. See
[A new repo appears](#p=new-repo) for the full lifecycle and [Skills](#p=skills) for what the
skill knows.

## `--suggest` — one-shot LLM reconciliation

Where `--skill` opens an interactive agent session, `--suggest` is a single structured call:
it sends the drift to the LLM, validates the returned JSON against a schema, and applies it
to **`view-map.local.json` only** — `assignments`, `newViews`, `ignore`. The server re-reads
the overlay on its next sync, so the proposal goes live without a restart and without
touching the deployment config. `--dry` prints instead of writing; `--exclude` fences off
scratch prefixes. The same mechanism runs server-side on every sync when `AUTO_RECONCILE=1`.

## Reading it in CI or scripts

`--once` renders a single frame and exits, so the CLI doubles as a check:

```bash
node cli.mjs --server http://localhost:8788 --once | grep '⚠ drift' && exit 1 || true
```
