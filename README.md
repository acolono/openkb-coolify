# coolify-openkbapp

[OpenKB](https://github.com/openkb-app/openkb) as a Coolify service template.
`templates/compose/openkb.yaml` is upstream's `docker-compose.yml` (1.x),
changed only where Coolify needs it (format of `coollabsio/coolify`, logo in
`svgs/openkb.svg`). Pinned to `1.0.0-beta2`.

| Service | Container port | Domain |
|---|---|---|
| `frontend` (Nuxt) | 3000 | `SERVICE_URL_FRONTEND`, the knowledge base |
| `drupal` (admin, API, OAuth) | 8080 | `SERVICE_URL_DRUPAL` |
| `cron`, `mariadb`, `opensearch` | internal | none |

## Deploy

Coolify generates both domains and every secret; the only required input is
the cookie domain.

1. Coolify → Project → **New Resource** → **Docker Compose Empty** → paste
   `templates/compose/openkb.yaml`.
2. **Environment variables**: set `SESSION_COOKIE_DOMAIN` to the parent domain
   of the two generated domains with a leading dot (e.g. `frontend-abc.example.com`
   and `drupal-abc.example.com` → `.example.com`). Optional: `OPENAI_API_KEY`,
   `MAILER_DSN`, `MAIL_FROM`.
3. **Deploy.** First boot installs Drupal (~5 min, 503 meanwhile). Sign in at
   the drupal domain `/user/login` as `admin` with `SERVICE_PASSWORD_ADMIN`
   (shown in the environment variables).
4. AI chat and semantic search need `OPENAI_API_KEY`. After setting it, in the
   drupal container: `drush search-api:reset-tracker kb_chunks -y`.

OpenSearch uses mmap as upstream does (`OPENSEARCH_ALLOW_MMAP=true`), which
wants `vm.max_map_count=262144` on the host (persist in `/etc/sysctl.d/`).
Without that sysctl, set `OPENSEARCH_ALLOW_MMAP=false`. Plan for ~4 GB RAM.

The template is not in Coolify's catalog: Coolify asks for 1,000 GitHub stars
upstream and has paused outside template contributions. Coolify only applies
its parsing to a pasted **Docker Compose Empty** resource, so test the
template there rather than with plain `docker compose`.

## Operating

* **Backups**: `openkb-drupal-files`, `openkb-collab-store` and `openkb-mariadb-data` must be backed
  up together. `openkb-opensearch-data` is rebuildable by reindexing.
* **Upgrades**: bump the image tags of `drupal`, `cron` and `frontend` in the
  template, redeploy. The image runs updatedb and config import on start.
* The Cloudron package of this app lives in `acolono/openkb-cloudron`.
