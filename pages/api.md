# HTTP API

## `GET /health`

```json
{ "ok": true, "gitlab": "https://gitlab.example", "group": "my-org/platform",
  "tokenSet": false, "live": true, "source": "files",
  "lastSync": "2026-07-08T07:00:00Z", "intervalSeconds": 30 }
```

`source` is `"gitlab"` or `"files"` — the same pipeline either way.

## `GET /live`

The whole state, rebuilt every sync:

| Field | Content |
| --- | --- |
| `views[]` | per view: expected repos with `found` flags and detected features |
| `unmapped[]` / `ignored[]` | drift: repos no view claims / deliberately ignored |
| `subscriptions` / `liveRegions` | sector × stage rollup and region codes, parsed from environment names |
| `apps` | tenants, namespaces, per-namespace clusters/regions |
| `network.subnets` | one entry per `subnet.<name>.tf` in the network repo |
| `security.kinds` | terraform security inventory: role assignments/definitions, subnets, firewall rules, NSG rules, peerings, private endpoints — each `{repo, file, names[]}` |
| `deployVars` | per repo: CI/CD variable NAMES defined + referenced (never values) |
| `signals` | deploy mechanism detection (Argo CD / Flux / pipelines) |
| `sourceLabel` / `sourceNote` / `drift` | presentation + optional external drift feed |

## `POST /ask`

`{ "question": "where are the permissions for a tenant given?" }` →
`{ "answer": "...", "sources": [{repo, url, file?}] }`.

Heuristic mode routes by view/knowledge/code search; with `ANTHROPIC_API_KEY` set the
answer is generated grounded in a compact snapshot of `/live` plus the matching concept
docs.
