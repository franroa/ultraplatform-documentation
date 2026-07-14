# A new repo appears — the drift lifecycle

What happens when somebody adds a repository to a platform the engine is already watching.
Nothing here is hypothetical — this exact sequence is re-verified against the showcase by
creating a throwaway repo and watching it surface.

## 1 · Detection is automatic

The engine re-syncs its source on every `--interval` (default 30s for the showcase). A new
repo — a new project in the GitLab group, or a new folder with a `.repo.json` in the files
tree — shows up in the next sync. No hook, no registration, no restart:

```
$ mkdir -p fake-platform/up-newthing && echo '{"description":"…","envs":[]}' > fake-platform/up-newthing/.repo.json
# ≤30s later:
$ curl -s :8788/live | jq .unmapped
["legacy/old-watchtower", "tools/rune-scratchpad", "up-newthing"]
```

The repo is inventoried immediately (features, terraform security scan, deploy variables,
environments → subscriptions). What it does **not** have yet is a place in the mapping —
so it lands in `unmapped[]`.

## 2 · Everything downstream warns at once

- the **CLI** footer: `⚠ drift: N unmapped repos · M expected-but-absent — run node cli.mjs --skill`
- the **map** shows its live-drift badge (bottom-left) while `/live` reports drift
- `--once` in a script exits you into the red path if you wired it that way

Drift is a first-class product state, not an error: the showcase deliberately ships two
unmapped strays so the detection is always demonstrable.

## 3 · Three ways to reconcile

| Path | What it edits | When |
| --- | --- | --- |
| **By hand** | `viewMap` in the deployment config (or the overlay's `ignore` list for scratch repos) | you know exactly where the repo belongs |
| **`cli.mjs --suggest`** | `view-map.local.json` overlay only — schema-validated LLM proposal, applied on the server's next sync | routine assignments, no restart wanted |
| **`cli.mjs --skill`** | an interactive `claude` session with the **improve-platform-map** skill and the drift summary | the drift is structural — a new view, a new region, something the 3D map must learn to draw |

## 4 · The skill path, step by step

`--skill` hands the agent the drift and the skill's *live drift workflow* prescribes:

1. read `/live` — `unmapped[]`, per-view `found:false`, `liveRegions` vs the map's regions;
2. put each relevant repo into its view in the deployment config (real names allowed **only**
   in the private companion's config), or into the overlay ignore list;
3. mirror the SHAPE of the change into the public showcase — the aliased
   `example.config.json` + a matching folder in the fake-platform tree — so the showcase
   keeps resembling the real platform;
4. if the drift is structural (a new region, a new view), extend the visualization through
   its data structures — region diagrams are generated, so a new region is a data-only change;
5. re-run `node cli.mjs --once` → the footer is clean (or reduced to the intentional ignore
   list), and both instances are compared side by side.

## 5 · What "reconciled" means

A repo is reconciled when a view claims it (`viewMap` or overlay `assignments`), or the
overlay `ignore` list names it as deliberate. Both are code/config — reviewable, revertable,
and re-read on every sync. There is no hidden state: delete the overlay and the drift
warnings come honestly back.
