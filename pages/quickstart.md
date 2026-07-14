# Quickstart

## 1 · Run the showcase (no credentials)

```bash
git clone <your copy of the engine>
server/demo.sh          # → http://localhost:8788
```

This serves the full experience against the bundled **fake platform**
(`server/config/fake-platform/` — ~30 fictional repos as real terraform/yaml/markdown
files). Edit any file in there and watch the map follow on the next 30-second sync.

## 2 · Point it at YOUR GitLab group

```bash
cp server/config/example.config.json my-platform.config.json
# edit: group, gitlabUrl, sectors, viewMap … and DELETE platformDir (that selects the files source)

GITLAB_TOKEN=<read_api PAT> CONFIG=my-platform.config.json PORT=8787 node server/server.mjs
```

Open `http://localhost:8787` — the map now renders your group. Expect drift warnings
first: unmapped repos are the engine telling you which views to define. See
[Configuration](#p=configuration).

## 3 · Or describe a platform as files

No GitLab? Export your inventory as a directory (one folder per repo + a small
`.repo.json` for metadata) and set `platformDir` in the config — the same pipeline runs.
Format in [Sources](#p=sources); a complete worked example (with the real outputs) in
[Onboarding a new platform](#p=new-platform).

## 4 · Ground the chat (optional)

Set `ANTHROPIC_API_KEY` on the server and `/ask` switches from heuristics to a grounded
LLM that answers from the live state and your docs. Set `AUTO_RECONCILE=1` and new drift
triggers a structured LLM suggestion merged into the view-map overlay.
