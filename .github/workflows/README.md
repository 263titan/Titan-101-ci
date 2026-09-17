# GitHub Actions Workflows — RETIRED

As of the Hub-and-Spoke re-architecture, `263titan/Titan-101-ci` no longer
runs **any** workflows that check out the private `Titan-101` monorepo. The
poller pipeline (`auto-ci.yml` → `pipeline.yml`), `e2e.yml`, `security.yml`,
the `*release*.yml` pipelines and the manual helpers were retired to enforce a
strict IP boundary: no proprietary source code, build logs, or intermediate
artifacts may be exposed to public runners.

The full workflow directory was removed with commit `c2701f2` unwound.

- **Hub (private `Titan-101`):** all proprietary builds, E2E/hardware tests,
  security scanning, and release pipelines run on a **self-hosted** runner
  (`runs-on: [self-hosted, titan-hub]`) — zero billing impact and secrets
  never leave the private repo.
- **Spokes (public, free runners):** PR/community validation and isolated
  artifact/test publishing on `titan-docs-portal`, `titan-plugins`,
  `titan-installer`, and `titan-sdk`. Triggered outward from the hub via
  `repository_dispatch` (hub → spoke only).

If this directory ever gains workflows again, they must operate exclusively
on this repo's own public code and must never access `Titan-101` production
secrets or checkout the private monorepo.