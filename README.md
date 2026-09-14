# Titan-101 CI Runner

This repository exists for one purpose: **run CI for the private `263titan/Titan-101` monorepo on GitHub's free, unlimited public-repo minutes.** It contains no application source code.

## How it works

```
+---------------------+        repository_dispatch (run-ci)         +---------------------+
|  Titan-101 (private)| ───────────────────────────────────────────► │  Titan-101-ci       |
|   .github/workflows |   payload: { repo, branch, sha, before }     │  (public runner)    │
|   ci-dispatch.yml   |                                             │                     │
+---------------------+                                             │  checks out private │
        ▲                                                           │  code (in-memory)  │
        │          commit status (ci/titan101-*)                    │  runs full CI       │
        └────────────────────────────────────────────────────────── ▼  reports status    │
                                                                    +---------------------+
```

1. A push to `Titan-101` triggers the tiny `ci-dispatch.yml` workflow (a few seconds of private minutes).
2. It sends a `repository_dispatch` event to this repo with the branch, commit SHA, and previous SHA.
3. This workflow checks out the private source directly into the runner workspace (never committed, never persisted — the VM is destroyed after each run), then runs the exact same pipeline as the old in-repo CI:
   - Install Dependencies → Detect Affected Projects → Build (TS) → Build (Rust Engines) → Unit Tests → Lint & Type Check.
4. Every job reports a `ci/titan101-*` commit status back to the private repo, so branch protection can require them.

## Required secrets

### On this repo (`Titan-101-ci`)

| Secret | Purpose |
|---|---|
| `REPO_ACCESS` | Fine-grained PAT: **Read** access to `Titan-101` repository contents + **Read/Write** commit statuses on `Titan-101`. (A classic PAT with the `repo` scope also works.) Used to check out the private code and to write statuses. |
| `CODECOV_TOKEN` *(optional)* | Coverage upload, if you use Codecov. |

### On the private repo (`Titan-101`)

| Secret | Purpose |
|---|---|
| `DISPATCH_TOKEN` | Fine-grained PAT: **Read/Write** access to issue/runs / repository_dispatch on this repo (or classic PAT with `repo` scope). Used by `ci-dispatch.yml` to fire the `run-ci` event. |

## Security notes

- The private source lives only in the runner VM's ephemeral workspace and is destroyed when the run finishes. It is never written into this repository's git history.
- Never put the `REPO_ACCESS` token anywhere except GitHub Actions secrets.
- The runner's own `GITHUB_TOKEN` is scoped to this public repo and cannot read the private repo — that is why `REPO_ACCESS` is required.

## Testing locally

Trigger a manual dispatch on the latest main SHA of `Titan-101`:

```bash
gh api repos/263titan/Titan-101/dispatches -f event_type=run-ci \
  -f client_payload="{\"repo\":\"263titan/Titan-101\",\"branch\":\"main\",\"sha\":\"<top-of-main>\",\"before\":\"<parent-sha>\"}"
```

To avoid consuming private minutes on every commit while iterating, temporarily switch `ci-dispatch.yml`'s `on` block to `workflow_dispatch` only.