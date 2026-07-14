# Onboarding a NEW platform

The design promise: **everything installation-specific lives in one JSON config**, so a
brand-new platform acquires the entire design — sync pipeline, views, drift, subscriptions,
security inventory, deploy variables, signals, chat — without touching engine code. This
page is the runbook, and it is *proven*: the walkthrough below was executed against a
from-scratch fictional platform and the outputs shown are real.

## The 10-minute walkthrough (files source)

**1 · Describe the platform as a directory.** One folder per repo (nesting allowed), a
`.repo.json` per repo, a `.group.json` at the root:

```
acme-platform/
  .group.json                       # { "variables": ["ARM_TENANT_ID","ARM_CLIENT_ID"] }
  payments/core-api/
    .repo.json                      # { "description":"…", "envs":["pay-live-we01","pay-live-ne01","pay-sandbox-we01"], "variables":["API_TIER"] }
    .gitlab-ci.yml                  # references $API_TIER and $MODEL_X
    charts/api/Chart.yaml           # ⇒ helm feature
    terraform/rbac.tf               # a role assignment ⇒ security inventory
  payments/ledger/.repo.json        # left out of the viewMap on purpose
  infra/network/
    .repo.json
    terraform/hub/subnet.gateway.tf # ⇒ subnet inventory
    terraform/hub/subnet.bastion.tf
  infra/identity/terraform/roles.tf # another role assignment
```

**2 · One config.**

```json
{
  "group": "acme/platform",
  "platformDir": "…/acme-platform",
  "sectors": ["pay"],
  "viewMap": [
    { "view": "Applications",      "repos": ["payments/core-api"] },
    { "view": "Network",           "repos": ["infra/network"] },
    { "view": "Identity & Access", "repos": ["infra/identity"] }
  ],
  "networkRepo": "infra/network",
  "sourceLabel": "ACME"
}
```

**3 · Run.**

```bash
CONFIG=acme.config.json PORT=8790 node server/server.mjs
```

**4 · What you get on first sync** (actual output of this walkthrough):

- `views[]` — all three views resolve, every repo `found:true` with detected features
  (`helm`+`ci` on core-api, `terraform` on network/identity);
- `unmapped: ["payments/ledger"]` — drift detection works from minute one;
- `subscriptions` — `pay-live` in `ne01`+`we01`, `pay-sandbox` in `we01`, parsed from the
  environment names; `liveRegions: ["ne01","we01"]`;
- `security.kinds.roleAssignments` — both role assignments, attributed to
  `infra/identity/terraform/roles.tf` and `payments/core-api/terraform/rbac.tf`;
- `network.subnets` — `hub/gateway`, `hub/bastion` from the `subnet.<name>.tf` convention;
- `deployVars` — core-api *defines* `API_TIER` but *references* `API_TIER`+`MODEL_X`:
  the dangling variable is visible before the first deploy ever runs;
- `signals.deployMechanism` — "GitLab CI pipelines" (drop an Argo CD `Application` manifest
  anywhere in the tree and it flips on the next sync).

Identical steps with a real group: delete `platformDir`, set `gitlabUrl`, export a
`read_api` `GITLAB_TOKEN`, run. Same pipeline, same outputs.

## Repo addressing (matters for nested repos)

Repos are addressed by their **group-relative path** (`payments/core-api`), everywhere:
`viewMap`, `networkRepo`, `appSources`, the overlay. A bare repo slug (`core-api`) is
accepted as a fallback for top-level repos. (This rule is enforced in the engine — it was
tightened after this very walkthrough exposed that nested `networkRepo` paths didn't
resolve; the fix keeps old slug-style configs working.)

## What transfers automatically vs what you author

| You get for free (config only) | You author (or generate) |
| --- | --- |
| sync pipeline, both sources | the 3D map's menu groups and hand-drawn `DETAIL` diagrams |
| views + found flags + features | the docs vault (`docs/*.md` concept notes) |
| drift: unmapped / absent / ignore | the region list the UI draws (`REGIONS[]` — data-only) |
| subscriptions from env names | the guided tour steps |
| terraform security inventory | |
| subnet + deploy-vars inventories | |
| deploy-mechanism signals | |
| `/ask` chat (heuristic; grounded with a key) | |
| CLI, overlay, `--suggest`, `--skill` | |
| generated region/AKS diagrams (fed by `/live`) | |

The right-hand column is exactly what the **improve-platform-map skill, Mode B** generates
from a GitLab group via the Knowledge Graph (gkg/Orbit): classify projects into
layers/views, derive scopes and resources, emit the data structures, write the notes. See
[Skills](#p=skills).

## Running several platforms side by side

One engine, N deployments — each is just a config + a port + its own overlay/state paths:

```bash
CONFIG=a.config.json PORT=8790 OVERLAY_PATH=a.overlay.json STATE_CACHE=a.state.json node server/server.mjs
CONFIG=b.config.json PORT=8791 OVERLAY_PATH=b.overlay.json STATE_CACHE=b.state.json node server/server.mjs
```

The showcase (:8788, files source) and the private live companion (:8787, gitlab source)
are literally this pattern — the same engine binary-for-binary, differing only in config.

## The honesty rule

If you maintain a public showcase next to a private deployment, every structural change on
the private side gets an anonymized counterpart in the showcase's file tree — same shapes,
fictional names — so the public demo always *resembles* the real platform, produced by the
same code. The publish step enforces the rest: a leak gate over every artifact, and a
second de-aliasing gate when the UI is synced into the private deployment.
