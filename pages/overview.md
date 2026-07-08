# Overview

**Ultraplatform** is a platform-engineering cockpit in three ideas:

1. **A PKM.** Every view, resource and article is a linked markdown note; the interactive
   3D map is how you browse the knowledge graph. Click anything — its note opens in a
   side panel, Notion-style, and links steer the visualization.
2. **An interactive platform.** The map is *live*: an engine syncs a real platform
   inventory (repos, environments, security-relevant terraform, deploy variables,
   applications) and the UI renders that state — views, regions, subnets, drift.
3. **Context for creating platforms.** The engine, the view model and the docs are
   generic. Point them at your own platform and they become your map, your grounded chat
   and your paved road.

## Architecture

```
      inventory SOURCE                 ONE pipeline                    consumers
┌───────────────────────┐      ┌──────────────────────────┐     ┌──────────────────┐
│ gitlab: a real group  │      │ sync → views → regions   │     │ 3D UI (the map)  │
│          — or —       │ ───▶ │ security scan · deploy   │ ──▶ │ /live JSON       │
│ files: a directory of │      │ vars · apps · drift      │     │ /ask chat        │
│ tf/yaml/md describing │      │ reconcile (optional LLM) │     │ CLI (terminal)   │
│ any platform          │      └──────────────────────────┘     └──────────────────┘
└───────────────────────┘
```

- **One pipeline, two sources.** There is no demo/fixture code path: the public showcase
  is a *fictional platform described as real files*, run through the same engine.
- **No databases.** Persistent state is files: the platform tree (or GitLab), the
  markdown docs, a JSON view-map overlay and a last-good state cache.
- **Config over code.** Everything installation-specific — group, view map, sector names,
  knowledge facts — lives in one JSON deployment config.

Continue with the [Quickstart](#p=quickstart).
