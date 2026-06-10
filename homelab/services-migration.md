# Services Migration Guide

This guide contains the checklist and reference resources for migrating containers and application storage configurations across homelab hosts (e.g., from old server to a new OptiPlex host).

---

## 1. Services Migration Checklist

Use this checklist to identify which services require state backups vs. configuration backups.

| Service | Backup Required | Critical Paths & Action Items |
|---|:---:|---|
| **Jellyfin** | Yes | `/config/data/backups/`. Skip transient directories like cache, logs, and artwork metadata. |
| **qBittorrent**| No | No config backup needed if default settings are okay. Ensure download paths (`/data/movies`) are preserved. |
| **Navidrome**  | Yes | Backup the database inside `data/backup/` to retain play counts, ratings, and playlists. |
| **Dockge**     | No | Docker Compose configuration files (e.g., `stacks/` directory) are sufficient. |
| **Grafana**    | Yes | Export configurations and manually export custom dashboards as `.json`. |
| **Prometheus** | Partial | Backup configuration files only (`prometheus.yml`). System metrics do not need migration. |
| **Immich**     | Yes | Perform database backup (`pg_dump` via compose) and backup original photos/videos folder. |
| **NGINX Proxy**| Yes | Backup virtual hosts (`/etc/nginx/sites-available/` or custom reverse proxy manager configs). |

---

## 2. Recommended Migration Workflow Sequence

Follow these high-level phases during homelab node migrations to minimize system downtime:

1. **Configurations & Compose Backups**: Gather all docker-compose directories, environment profiles (`.env`), and backup databases for stateful services (Jellyfin, Immich, Navidrome).
2. **Target Host Initialization**: Install Debian on the new machine, configure [Docker & Docker Compose], and verify [Intel/NVIDIA GPU passthrough].
3. **Mount Storage**: Mount external drives, configure [ZFS Mirroring], and set up permanent system mount points.
4. **Deploy Stack**: Deploy your compose stacks with services in a stopped or paused state.
5. **Restore Backups**: Restore the configuration files and databases into the respective container directories.
6. **Start Stack & Verify**: Run `docker compose up -d` and inspect logs/permissions for validation.
