# actual-budget

Kubernetes deployment manifests for [Actual](https://actualbudget.org/), a self-hosted envelope-budgeting app. No build step here — consumes the upstream `actualbudget/actual-server` image directly, same shape as [`dvystrcil/invidious`](https://github.com/dvystrcil/invidious).

Licensed under [MIT](LICENSE).

## Storage

SQLite-only — confirmed by reading the real config schema (`packages/sync-server/src/load-config.js`): no Postgres option exists at all. This is a real exception to the [[sqlite-to-pgo-default]] convention, not an oversight. Data lives on a single `ReadWriteOnce` PVC (`server-files/account.sqlite` + one `db.sqlite` blob per budget file), so the Deployment uses `strategy: Recreate` to avoid a rolling-update deadlock over the single volume.

## Auth

No env var pre-seeds a password — confirmed in `app-account.js`: whoever visits first completes an interactive bootstrap flow (`POST /bootstrap`) that sets the server password. **First real visitor should be you, not anyone else on the network** — the instance isn't internet-exposed (LAN + Pi-hole local DNS only), but do the bootstrap step promptly after this deploys.

OIDC/Keycloak support exists upstream (Keycloak is a tested provider) but Keycloak isn't deployed live in this cluster yet — deferred until it is (homelab#847).

## Backup

No server-side automatic backups exist upstream (desktop-app-only feature, confirmed in docs). Backup story here is volume-level (PVC snapshot), not anything Actual itself does — factor the `actual-budget-data` PVC into whatever backup rotation covers other PVCs in this cluster.

## Structure

- `base/` — namespace, PVC, Deployment, Service, VPA (recommendation-only, D-012)
- `overlays/` — ArgoCD entry point

Deployed via [`dvystrcil/argocd-projects`](https://github.com/dvystrcil/argocd-projects)'s `actual-budget` Application.
