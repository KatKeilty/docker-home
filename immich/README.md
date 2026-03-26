# Immich

Self-hosted photo and video backup (Google Photos alternative).

## Access

https://immich.yourdomain.com

## What It Does

- Automatic photo/video backup from phone
- Face recognition
- Object detection (searchable)
- Albums and sharing
- Map view (location-based)
- Live photos support
- RAW format support
- Video transcoding

## Files

```
immich/
├── docker-compose.yml
├── .env
├── upload/                # Photos and videos (BACKUP THIS)
├── library/              # User libraries
├── profile/              # User avatars
├── thumbs/               # Thumbnails (can regenerate)
└── README.md
```

## docker-compose.yml

```yaml
name: immich
services:
  immich-server:
    container_name: immich-server
    image: ghcr.io/immich-app/immich-server:release
    command: ['start.sh', 'immich']
    volumes:
      - ./upload:/usr/src/app/upload
      - ./library:/usr/src/app/library
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    depends_on:
      - redis
      - database
    restart: unless-stopped
    networks:
      - app-network

  immich-microservices:
    container_name: immich-microservices
    image: ghcr.io/immich-app/immich-server:release
    command: ['start.sh', 'microservices']
    volumes:
      - ./upload:/usr/src/app/upload
      - ./library:/usr/src/app/library
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    depends_on:
      - redis
      - database
    restart: unless-stopped
    networks:
      - app-network

  immich-machine-learning:
    container_name: immich-machine-learning
    image: ghcr.io/immich-app/immich-machine-learning:release
    volumes:
      - ./model-cache:/cache
    env_file:
      - .env
    restart: unless-stopped
    networks:
      - app-network

  redis:
    container_name: immich-redis
    image: redis:7.2-alpine
    restart: unless-stopped
    networks:
      - app-network

  database:
    container_name: immich-postgres
    image: tensorchord/pgvecto-rs:pg14-v0.2.0
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
    volumes:
      - ./postgres:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - app-network

networks:
  app-network:
    external: true
```

## .env file

```env
# Database
DB_HOSTNAME=database
DB_USERNAME=immich
DB_PASSWORD=change-this-to-random-password
DB_DATABASE_NAME=immich

# Upload location
UPLOAD_LOCATION=./upload

# Timezone
TZ=America/New_York

# Redis
REDIS_HOSTNAME=redis

# Machine learning (can disable if low resources)
MACHINE_LEARNING_ENABLED=true
```

**Generate random password:**
```bash
openssl rand -base64 32
```

## First Time Setup

1. Access web interface
2. Create admin account
3. Download mobile app
4. Login on mobile
5. Enable backup in app settings
6. Photos upload automatically

## Mobile Apps

**iOS:** Immich (official)
**Android:** Immich (official)

**Setup:**
1. Install app
2. Enter server URL: `https://immich.yourdomain.com`
3. Login
4. Settings > Backup > Enable automatic backup
5. Choose which albums to backup
6. App backs up when charging/wifi (configurable)

## Features

### Face Recognition

1. Settings > Machine Learning > Face Recognition: Enable
2. System scans all photos
3. Click face thumbnails to name people
4. Search by person name

### Smart Search

Search for:
- Objects: "dog", "beach", "car"
- Places: "Paris", "mountains"
- People: Names you've tagged
- Dates: "2024"
- Combined: "beach with Sarah in 2023"

### Albums

- Create albums in web interface
- Add photos manually or by search
- Share albums with links (optional password)
- Shared users can view/download

### Live Photos

iOS Live Photos are supported (video + still).

### Map View

Photos with GPS metadata appear on map.

## Sharing

### Share album
1. Create album
2. Click share icon
3. Generate link
4. Optional: Set password, expiry
5. Anyone with link can view

### Share individual photo
1. Select photo
2. Click share
3. Copy link or download

## Backup Strategy

**Critical data:**
- `./upload/` - All original photos/videos

**Regenerable:**
- `./thumbs/` - Thumbnails
- `./model-cache/` - ML models
- `./postgres/` - Database (can rebuild from photos)

**Backup command:**
```bash
# Backup photos (the important part)
tar -czf immich-photos-$(date +%Y%m%d).tar.gz ./upload

# Full backup (includes database)
tar -czf immich-full-$(date +%Y%m%d).tar.gz ./upload ./postgres ./library ./profile
```

**Restore:**
```bash
docker compose down
tar -xzf immich-full-YYYYMMDD.tar.gz
docker compose up -d
```

## Maintenance

### Re-run face detection
Settings > Jobs > Face Detection > Run All

### Re-generate thumbnails
Settings > Jobs > Thumbnail Generation > Run All

### Storage analysis
Settings > Storage > Analyze usage

### Delete duplicate photos
Immich detects duplicates automatically.
Settings > Duplicates > Review and delete

## Troubleshooting

### Mobile app won't backup

- Check server URL is correct
- Check SSL certificate is valid
- Settings > Backup > Check "Background backup" is enabled
- iOS: Settings > Immich > Background App Refresh: On

### Face recognition not working

```bash
# Check ML container is running
docker ps | grep immich-machine-learning

# Check logs
docker logs immich-machine-learning

# Restart job
# Web UI > Settings > Jobs > Face Detection > Run All
```

### High CPU/memory usage

ML features are resource-intensive.

**Reduce load:**
- Disable face recognition (Settings > ML)
- Disable object detection (Settings > ML)
- Reduce thumbnail quality (Settings > Thumbnails)

**Or upgrade hardware:**
- ML works better with GPU (optional)

### Can't upload photos

```bash
# Check upload folder permissions
ls -la ./upload

# Check logs
docker logs immich-server
docker logs immich-microservices

# Restart services
docker compose restart
```

### Database issues

```bash
# Check database
docker logs immich-postgres

# Backup and recreate
docker exec immich-postgres pg_dump -U immich immich > backup.sql
docker compose down
rm -rf ./postgres/*
docker compose up -d
docker exec -i immich-postgres psql -U immich < backup.sql
```

## Performance Tips

- Upload original quality (Immich handles optimization)
- Let ML run overnight (face recognition is slow)
- Use SSD for `./upload` if possible
- GPU helps with ML but not required

## Updating

**IMPORTANT:** Always check release notes. Immich updates frequently.

```bash
cd immich
docker compose pull
docker compose up -d
```

Database migrates automatically on startup.

## Resource Usage

**Varies by workload:**
- **Idle:** 200-500MB RAM, <5% CPU
- **Uploading:** 500MB-1GB RAM, 10-30% CPU
- **ML processing:** 1-4GB RAM, 30-100% CPU (temporary)

## Privacy

- All photos stored locally on your server
- ML processing happens on-server (not cloud)
- No external services required
- Metadata stays private
- Optional sharing via links

## Compared to Google Photos

**Better:**
- Full control/ownership
- No storage limits (except disk)
- No compression/quality loss
- Privacy

**Not as good (yet):**
- ML features less mature
- No automatic movie creation
- Slower development

## Tips

- Let initial ML processing finish before judging performance
- Mobile app drains battery during initial upload (normal)
- Use wifi for uploads (can use cellular but slow)
- Organize with albums (like Google Photos)
- Share sparingly (each share is publicly accessible)