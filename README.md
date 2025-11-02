# Firefox Sync Server (syncstorage-rs)

Self-hosted Mozilla Firefox Sync Storage server running on Docker with MariaDB.

## Features

- Multi-architecture Docker image (amd64, arm64)
- MariaDB database backend
- Automated GitHub Actions CI/CD pipeline
- Non-root container user for security
- UTF-8 database character set
- Automatic database migrations

## File Structure

```
firefox-sync/
├── .github/
│   └── workflows/
│       └── docker-publish.yml
├── app/
│   ├── Dockerfile
│   └── entrypoint.sh
├── data/
│   └── initdb.d/
│       └── init.sql
├── docker-compose.yml
├── example.env
├── .gitignore
└── README.md
```

## Quick Start

### 1. Clone and Setup

```
git clone <your-repo>
cd firefox-sync
cp example.env .env
```

### 2. Generate Required Secrets

```
# Generate 64-character master secret
cat /dev/urandom | base32 | head -c64
# Copy this to SYNC_MASTER_SECRET in .env

# Generate 64-character metrics hash secret
cat /dev/urandom | base32 | head -c64
# Copy this to METRICS_HASH_SECRET in .env

# Generate 32-character passwords
cat /dev/urandom | base32 | head -c32
# Copy to MYSQL_ROOT_PASSWORD
# Copy to MYSQL_PASSWORD (for sync user)
```

### 3. Edit `.env`

```
nano .env

# Required fields:
SYNC_URL=https://sync.example.com  # Your public URL
SYNC_MASTER_SECRET=<generated-64-char-secret>
METRICS_HASH_SECRET=<generated-64-char-secret>
MYSQL_ROOT_PASSWORD=<generated-32-char-password>
MYSQL_PASSWORD=<generated-32-char-password>
```

### 4. Start the Services

```
docker compose up -d

# Check logs
docker compose logs -f syncserver

# Test the heartbeat
curl http://localhost:8000/__heartbeat__

# Expected response:
# {"version":"0.18.3","quota":{"enabled":false,"size":0},"database":"Ok","status":"Ok"}
```

### 5. Stop Services

```
docker compose down

# Keep data:
docker compose down

# Remove data:
docker compose down -v
```

## Configuration

### Environment Variables (`.env`)

| Variable | Description | Example |
|----------|-------------|---------|
| `SYNC_URL` | Public URL for clients | `https://sync.example.com` |
| `SYNC_CAPACITY` | Max concurrent users | `10` |
| `SYNC_MASTER_SECRET` | Encryption key (64 chars) | (generated) |
| `METRICS_HASH_SECRET` | Hashing key (64 chars) | (generated) |
| `MYSQL_ROOT_PASSWORD` | MariaDB root password | (generated) |
| `MYSQL_PASSWORD` | Sync user password | (generated) |
| `LOGLEVEL` | Logging level | `warn` |

## Connecting Firefox

### Firefox Desktop

1. Go to `about:config`
2. Set `identity.sync.tokenserver.uri` to `https://sync.example.com/token/1.0/sync/1.5`
3. Restart Firefox
4. Go to **Settings** → **Sync** and sign in

### Firefox Mobile

1. Tap menu → **Settings**
2. Tap **Sync**
3. Enter custom server: `https://sync.example.com`
4. Sign in

## Production Deployment

### Using GitHub Container Registry

```
# Pull the latest image
docker pull ghcr.io/youruser/firefox-sync:main

# Run with your .env file
docker compose up -d
```

### Manual Docker Build

```
# Build locally
docker compose build

# Test locally
docker compose up -d

# After merging to main branch, GitHub Actions will automatically build and push to ghcr.io
```

## GitHub Actions Workflow

The repository includes a CI/CD pipeline that:

1. **On push to `dev`**: Builds and tests the image
2. **On push to `main`**: Builds multi-arch images (amd64, arm64) and pushes to GitHub Container Registry
3. **On version tags** (`v*.*.*`): Creates release images
4. **Manual trigger**: Allows building specific architectures via `workflow_dispatch`

### Manual Workflow Trigger

```
# Push to dev first to test
git push origin dev

# Once tested, merge to main
git checkout main
git merge dev
git push origin main

# This triggers the automated build and push to ghcr.io
```

## Monitoring

### View Logs

```
# Real-time logs
docker compose logs -f syncserver

# Last 50 lines
docker compose logs --tail=50 syncserver

# Mariadb logs
docker compose logs -f mariadb
```

### Health Check

```
# Server health
curl http://localhost:8000/__heartbeat__

# Database test
docker compose exec mariadb mysql -u root -p${MYSQL_ROOT_PASSWORD} -e "SELECT 1;"
```

## Troubleshooting

### Container won't start

```
# Check logs
docker compose logs syncserver

# Rebuild
docker compose build --no-cache

# Restart
docker compose restart syncserver
```

### Database connection errors

```
# Verify database is running
docker compose ps mariadb

# Check database credentials in .env
grep MYSQL_ .env

# Test connection
docker compose exec mariadb mysql -u sync -p${MYSQL_PASSWORD} -e "SELECT 1;"
```

### Port already in use

```
# Change ports in docker-compose.yml
# Modify the ports section:
# ports:
#   - "8001:8000"  # Changed from 8000:8000
```

## Updating

### Update to Latest Code

```
# Pull latest changes
git pull origin dev

# Rebuild image
docker compose build

# Restart with new image
docker compose up -d
```

## Security Notes

- Always use HTTPS in production with a reverse proxy (nginx, Cloudflare)
- Never expose the server directly to the internet on HTTP
- Rotate `SYNC_MASTER_SECRET` and `METRICS_HASH_SECRET` periodically
- Use strong, randomly generated passwords
- Keep Docker and dependencies updated

## Backup

### Database Backup

```
docker compose exec mariadb mysqldump -u sync -p${MYSQL_PASSWORD} syncstorage_rs > backup.sql
docker compose exec mariadb mysqldump -u sync -p${MYSQL_PASSWORD} tokenserver_rs >> backup.sql
```

### Restore Backup

```
docker compose exec -T mariadb mysql -u sync -p${MYSQL_PASSWORD} < backup.sql
```

## Resources

- [syncstorage-rs GitHub](https://github.com/mozilla-services/syncstorage-rs)
- [Firefox Sync Documentation](https://github.com/mozilla-services/syncstorage-rs)
- [Docker Documentation](https://docs.docker.com)

## License

This project is licensed under the MPL-2.0 License (same as syncstorage-rs).
```

## Setup Instructions Summary

### Step 1: Initial Setup

```bash
git clone <your-repo>
cd firefox-sync
cp example.env .env
```

### Step 2: Generate Secrets

```bash
# Generate all required secrets and paste into .env
cat /dev/urandom | base32 | head -c64  # For SYNC_MASTER_SECRET
cat /dev/urandom | base32 | head -c64  # For METRICS_HASH_SECRET
cat /dev/urandom | base32 | head -c32  # For passwords
```

### Step 3: Configure

```bash
nano .env
# Update: SYNC_URL, SYNC_MASTER_SECRET, METRICS_HASH_SECRET, passwords
```

### Step 4: Test Locally

```bash
docker compose build
docker compose up -d
docker compose logs -f syncserver
curl http://localhost:8000/__heartbeat__
```

### Step 5: Deploy

```bash
git add .
git commit -m "Initial deployment configuration"
git push origin dev

# After testing, merge to main
git checkout main
git merge dev
git push origin main

# GitHub Actions automatically builds and pushes to ghcr.io
```
