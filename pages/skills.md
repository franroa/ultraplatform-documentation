# Agent skills

The repetitive, error-prone procedures around this project are encoded as **agent skills** —
versioned markdown playbooks an AI agent (Claude Code) loads on demand. They live in
`skills/` in the source repo and are exposed as a plugin marketplace entry, so any session
— including one launched *by the CLI itself* — works from the same paved road. The thesis
is the golden-path idea applied to agents: the skill is the documented, reviewed way, and
it is also the easiest way.

## improve-platform-map

The umbrella skill for everything map/engine/site. Two operating modes:

- **A · Maintain** — add or edit a view, 3D diagram, resource note, tour step, region,
  blog/notes/terminal content on the existing map. Encodes the data model (`MENU`,
  `DETAIL`, `RES`, `REGIONS`, scope bands…), the anonymization rule with its leak gates,
  mobile conventions, the verification matrix (smoke test, headless behavioral checks,
  dual-instance comparison), and the publishing pipeline.
- **B · Generate** — build a whole new map for ANY GitLab group by introspecting it through
  the GitLab Knowledge Graph (gkg locally, Orbit remotely): classify projects into layers
  and views, derive scopes/resources/repos-by-level, emit the data structures and notes,
  wire the chat. This is the answer to "what if I point all this at a different platform" —
  see [Onboarding a new platform](#p=new-platform).

Four reference playbooks ride inside it:

| Reference | Covers |
| --- | --- |
| `live-drift-reconcile` | the drift lifecycle: triggers (CLI footer, map badge, `cli.mjs --skill`), the 5-step reconciliation, the terraform security cross-check |
| `generate-from-gitlab-group` | Mode B end-to-end: engines, indexing, classification, emission |
| `dual-instance-comparison` | the always-run gate: showcase vs live companion, four parity checks |
| `worked-examples` | the same change (region, subnet, policy, tenant permission) done on both sides |

## write-post

The publishing skill for the Writing page. It encodes the house format (emoji H1, TL;DR
callout, numbered sections, bare-key cross-links), the **4-place registration** (`POSTS[]`,
blog index, map menu node, regenerators), the rule that platform posts must link into the
map (with an explicit opt-out set for personal-tooling essays), leak safety for public
content, and **private posts** — articles that render only on the author's machine and can
never reach a public artifact by construction.

## sync-landing-sections

The closing-move skill for content sessions. The landing page summarizes the platform in
three places — the stats band under the globe (baked numbers + runtime refresh from the
live snapshot), the client-side role-fit matcher (keyword areas, each linking to proof),
and the for-recruiters card (authored once, cloned over the globe at runtime) — and none of
them updates itself when the fake platform grows, a post lands, or a map lens ships. This
skill is the checklist that re-syncs all three plus the documentation pages stating the
same facts, with the verification steps to prove they agree before publishing.

## How the CLI hands work to a skill

`node cli.mjs --skill` is the integration point: it fetches `/live`, summarizes the drift,
and launches the agent **with the improve-platform-map skill preloaded** and the summary as
the prompt. The agent then follows the same reference playbook a human would — which is the
point: the runbook and the automation are one artifact, so they cannot drift apart.

## Why skills instead of scripts

- **Judgment steps stay explicit.** "Does this repo belong to the Network view or the
  ignore list?" is a decision, not a regex — the skill frames it, the agent (or you) makes it.
- **Guardrails travel with the task.** The anonymization rules, the two leak gates, the
  never-hand-edit-generated-files rule are in the skill text, so every session re-reads them.
- **They version like code.** Each skill carries a semver and changes land in review next
  to the code they describe; when a session teaches a hard lesson, the lesson is folded
  back into the skill the same day.
