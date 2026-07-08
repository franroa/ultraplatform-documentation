# Sources — gitlab | files

The engine reads its raw inventory through one interface with two implementations
(`server/sources.mjs`). Everything downstream — views, subscriptions, security inventory,
apps, chat — is identical.

## gitlab

The real thing: lists the group's projects (subgroups included), reads trees/files for
feature detection and app collection, environments for the sector×stage×region model,
group blob search for the terraform security inventory, CI/CD **variable names** (never
values). Needs a `read_api` token.

## files

A directory *is* a platform:

```
my-platform/
  .group.json                 # { "variables": ["ARM_TENANT_ID", …] }
  team-a/service-x/           # any nesting — a repo is a folder containing .repo.json
    .repo.json                # { "description": …, "envs": ["core-live-eu01"], "variables": […] }
    .gitlab-ci.yml            # variables referenced here are collected
    versions.tf               # *.tf at the root ⇒ terraform feature; scanned for the
    terraform/…               #   security inventory (role assignments, subnets, …)
    charts/…                  # ⇒ helm feature
    README.md                 # feeds the chat's repo answers
```

`.repo.json` holds only what GitLab would hold *outside* the tree: description,
environment names, variable names. Everything else — features, subnets, security
inventory, namespaces, tenants — is derived from the actual files, exactly like the
gitlab source derives it from the real repos.

## The bundled fake platform

`server/config/fake-platform/` is a complete fictional platform in this format: it exists
so the public showcase runs the LIVE pipeline instead of a fixture, and it doubles as the
reference example for the files format. Deliberate imperfections included — an unmapped
legacy repo, an expected-but-absent one — so drift detection has something to show.
