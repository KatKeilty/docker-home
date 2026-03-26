# docker-home

A collection of Docker Compose files for my self-hosted home stack, running on a QNAP TS-264 NAS.

Each subdirectory is a standalone service with its own `compose.yml`.

## Services

| Service | Purpose |
|---|---|
| [actual-budget](./actual-budget/) | Personal finance and budgeting |
| [gitea](./gitea/) | Self-hosted Git with SSH and web access |
| [grocy](./grocy/) | Household and pantry management |
| [immich](./immich/) | Photo and video backup |
| [mealie](./mealie/) | Recipe management and meal planning |
| [nextcloud](./nextcloud/) | File sync, documents, and collaboration |
| [paperless](./paperless/) | Document scanning and archiving |
| [portainer](./portainer/) | Docker container management UI |

## Usage

```bash
cd <service>
docker compose up -d
```

## Notes

- Reverse proxy handled by Caddy (separate config)
- Private networking via Tailscale
- DNS managed through Cloudflare

---

# docker-home

Une collection de fichiers Docker Compose pour mon infrastructure personnelle auto-hébergée, fonctionnant sur un NAS QNAP TS-264.

Chaque sous-dossier est un service autonome avec son propre fichier `compose.yml`.

## Services

| Service | Utilité |
|---|---|
| [actual-budget](./actual-budget/) | Finances personnelles et budget |
| [gitea](./gitea/) | Git auto-hébergé avec accès SSH et web |
| [grocy](./grocy/) | Gestion du foyer et du garde-manger |
| [immich](./immich/) | Sauvegarde de photos et vidéos |
| [mealie](./mealie/) | Gestion de recettes et planification des repas |
| [nextcloud](./nextcloud/) | Synchronisation de fichiers, documents et collaboration |
| [paperless](./paperless/) | Numérisation et archivage de documents |
| [portainer](./portainer/) | Interface de gestion des conteneurs Docker |

## Utilisation

```bash
cd <service>
docker compose up -d
```

## Notes

- Proxy inverse géré par Caddy (configuration séparée)
- Réseau privé via Tailscale
- DNS géré par Cloudflare
