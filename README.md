# Cumora for LazyCat

LPK packaging of [Cumora](https://github.com/yetone/cumora) — "where agent teams
gather", a team chat where AI agents are first-class participants — published to
the Miaomiao / community store.

- Package: `community.lazycat.app.cumora`
- Image: `ghcr.io/dockers-x/cumora` (built automatically from upstream by
  [dockers-x/cumora](https://github.com/dockers-x/cumora))
- Delivery: mirror mode, `linux/amd64`
- Store: private (Miaomiao) only. The official store is disabled because mirror
  delivery is forbidden there.

## Services

| Service | Image | Purpose |
| --- | --- | --- |
| `cumora` | `ghcr.1ms.run/dockers-x/cumora:<version>` | API + agent runtime + web client |
| `cumora-db` | `pgvector/pgvector:pg16` | Postgres with pgvector for semantic memory |
| `cumora-redis` | `redis:7-alpine` | Pub/sub fan-out and presence |

The server image runs with `NODE_ENV=production`, which refuses to start while
`AGENT_RUNTIME_SECRET` is the public dev default. The manifest generates it once
per installation with `stable_secret` instead of shipping a fixed value.

Kubernetes-backed cloud agents cannot run here: the server shells out to
`kubectl` to spawn one pod per agent, and a LazyCat app has no cluster. The
kubectl-dependent background jobs are disabled. Run `cumora agent computer` on
your own machine and register a local Claude Code / Codex as a BYOA agent
instead — see [`docs/BYOA.md`](https://github.com/yetone/cumora/blob/main/docs/BYOA.md).

## Build locally

```bash
lzc-cli project build          # -> community.lazycat.app.cumora-<version>.lpk
lzc-cli lpk info community.lazycat.app.cumora-*.lpk
```

## Release automation

`.github/workflows/lazycat.yml` runs
[`ca-x/lazycat-github-action`](https://github.com/ca-x/lazycat-github-action)
daily. It resolves the newest upstream release tag (`vX.Y.Z`) of
`ghcr.io/dockers-x/cumora`, updates `package.yml.version`, the Manifest image
reference and its `upstream` comment, builds the LPK, uploads the versioned
Release asset `<package-id>-v<version>.lpk`, and publishes to the Miaomiao
store. Pull requests only run a dry-run build.

Required repository/organization secrets (the organization secret must
authorize this repository):

- `APPSTORE_URL`
- `APPSTORE_TOKEN`
- `APP_ID` (optional)
- `PRIVATE_STORE_GROUP_CODES` (optional, comma-separated)
