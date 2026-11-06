# coolify-openkbapp

[OpenKB](https://github.com/openkb-app/openkb) on Coolify. `docker-compose.yml`
is upstream's production compose file (1.x), adapted: published images only,
no host ports, domains assigned in Coolify. Pinned to `1.0.0-beta2`.

| Service | Container port | Domain |
|---|---|---|
| `frontend` (Nuxt) | 3000 | primary, e.g. `kb.example.com` |
| `drupal` (admin, API, OAuth) | 8080 | admin, e.g. `kb-admin.example.com` |
| `cron`, `mariadb`, `opensearch` | internal | none |

## Deploy

1. Coolify → Project → **New Resource** → Public/Private Repository (Git over
   SSH with a deploy key) → this repo, branch `main`, build pack
   **Docker Compose**, compose file `/docker-compose.yml`.
2. **Domains**: set `frontend` to `https://kb.example.com:3000` and `drupal`
   to `https://kb-admin.example.com:8080`. The `:port` tells Traefik which
   container port to route to.
3. **Environment variables**: copy `.env.example`, fill in the secrets
   (`openssl rand -hex 32`) and make the URLs match step 2. Both hostnames
   must share the parent domain in `SESSION_COOKIE_DOMAIN`.
4. **Host**: either `sysctl -w vm.max_map_count=262144` (persist in
   `/etc/sysctl.d/`), or set `OPENSEARCH_ALLOW_MMAP=false`. Plan for ~4 GB RAM.
5. **Deploy.** First boot installs Drupal (~5 min; Drupal answers 503 with a
   setup page meanwhile). Sign in at `https://<admin-domain>/user/login` as
   `admin` (`ADMIN_PASSWORD`, or read the generated one from the drupal logs).
6. AI chat and semantic search need `OPENAI_API_KEY`. After setting it, in the
   drupal container: `drush search-api:reset-tracker kb_chunks -y`.

## Operating

* **Backups**: `drupal-files`, `collab-store` and `mariadb-data` must be backed
  up together. `opensearch-data` is rebuildable by reindexing.
* **Upgrades**: bump `OPENKB_TAG`, redeploy. The image runs updatedb and config
  import on start.
* The Cloudron package of this app lives in `acolono/openkb-cloudron`.
