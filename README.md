# unified-ip-api

Build home for the **Unified IP Information API** — an API-only IP information
service assembled from two upstream projects:

| Piece | Upstream | Licence | Built from |
|---|---|---|---|
| Main image | [jason5ng32/MyIP](https://github.com/jason5ng32/MyIP) via the fork [`ChinaToyHunter/MyIP`](https://github.com/ChinaToyHunter/MyIP) | MIT | repo-root `Dockerfile` |
| Egress-probe sidecar | [xykt/IPQuality](https://github.com/xykt/IPQuality), pinned by commit SHA and verified by sha256 | AGPL-3.0 | `ipquality-sidecar/` |

The sidecar stays a separate image on purpose: AGPL-3.0 code is kept out of the
MIT image and reached over HTTP, never linked into it.

## Why this repository exists

The source of truth is the fork's feature branch (`feat/unified-v1`), not this
repository. Build (and later deploy) concerns live here so the fork carries
nothing but upstream history plus its own feature branch — a fork-only workflow
file in the fork would be a merge conflict on every upstream sync.

## Images

| Image | GHCR |
|---|---|
| Main | `ghcr.io/chinatoyhunter/myip` |
| Sidecar | `ghcr.io/chinatoyhunter/myip-ipquality-sidecar` |

Tags:

- `latest`
- `sha-<12>` — the **source** commit in the fork that was compiled (not this
  repository's commit)

Both are `linux/amd64`.

## Running a build

Actions → **Build images** → *Run workflow*, or:

```bash
gh workflow run images.yml --repo ChinaToyHunter/unified-ip-api -f ref=feat/unified-v1
```

Pushing to this repository's `main` also builds. **No secrets are required** —
both repositories are public, and the GHCR push uses the workflow's own
`GITHUB_TOKEN`. The sidecar job boots the built image and hits `/healthz` before
publishing, so a broken image never reaches the registry.

## Deployment notes

Not wired up yet. Two things to know when it is:

- **GHCR packages are private by default**, even from a public repository.
  Either `docker login ghcr.io` on the host or flip the package's visibility to
  public.
- **No credentials are needed to build, but several are needed to run.** Provider
  API keys (`IPINFO_API_KEY`, `IP2LOCATION_API_KEY`, `IPDATA_API_KEY`, …),
  `UNIFIED_API_KEYS` for the key-gated routes, `IPQUALITY_SIDECAR_URL` pointing
  at the sidecar, and either MaxMind credentials or a mounted
  `common/maxmind-db/`. The GeoLite2 databases are deliberately excluded from the
  build context — a container downloads them on first boot.

The API surface itself is documented in the fork at
[`UNIFIED_API.md`](https://github.com/ChinaToyHunter/MyIP/blob/feat/unified-v1/UNIFIED_API.md).
