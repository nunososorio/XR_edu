# Docker Deployment Guide for XR Educational Platform

## Overview

This guide provides detailed Docker deployment instructions for the XR Educational Bookmark Platform built on Linkwarden.

## Docker Architecture

### Current Setup with Docker Compose

The `docker-compose.yml` orchestrates three services:

```
┌─────────────────────────────────────────────────────────┐
│         XR Educational Bookmark Platform                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Linkwarden │  │  PostgreSQL  │  │ MeiliSearch  │  │
│  │    (Web)     │  │   (Database) │  │   (Search)   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│     Port: 3000      Port: 5432         Port: 7700      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Services Description

#### 1. Linkwarden Web App
- **Image:** `ghcr.io/linkwarden/linkwarden:latest`
- **Port:** 3000
- **Purpose:** Main application interface
- **Volumes:** 
  - `./data:/data/data` - Persistent file storage
  - `./.env` - Configuration file

#### 2. PostgreSQL Database
- **Image:** `postgres:16-alpine`
- **Port:** 5432 (internal)
- **Purpose:** Persistent data storage
- **Volumes:** 
  - `./pgdata:/var/lib/postgresql/data` - Database files

#### 3. MeiliSearch
- **Image:** `getmeili/meilisearch:v1.12.8`
- **Port:** 7700 (internal)
- **Purpose:** Full-text search capabilities
- **Volumes:**
  - `./meili_data:/meili_data` - Search index

## Production Deployment

### Before Going Live

#### 1. Security Hardening

**Environment Variables:**
```env
# CRITICAL: Generate secure secret
NEXTAUTH_SECRET=your-output-from-openssl-rand-base64-32

# Change database password
POSTGRES_PASSWORD=your-secure-password

# Change database credentials in connection string
DATABASE_URL=postgresql://data_user:your-secure-password@postgres:5432/linkwarden
```

**Generate secure secrets:**
```bash
# PowerShell
[Convert]::ToBase64String([System.Security.Cryptography.RandomNumberGenerator]::GetBytes(32))

# Or use OpenSSL
openssl rand -base64 32
```

#### 2. Update PostgreSQL Credentials

Modify `.env`:
```env
# Old (development)
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/linkwarden
POSTGRES_PASSWORD=postgres

# New (production)
DATABASE_URL=postgresql://linkwarden_user:SECURE_PASSWORD@postgres:5432/linkwarden
POSTGRES_PASSWORD=SECURE_PASSWORD

# Consider creating separate read-only user for backups
```

#### 3. Configure Reverse Proxy

For HTTPS and domain access, use nginx:

**Create `nginx.conf`:**
```nginx
upstream linkwarden {
    server linkwarden:3000;
}

server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL certificates (use Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Proxy settings
    proxy_pass http://linkwarden;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Performance
    client_max_body_size 500M;
    proxy_read_timeout 300s;
    proxy_connect_timeout 75s;
}
```

**Add to docker-compose.yml:**
```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certs:/etc/letsencrypt:ro
    depends_on:
      - linkwarden

  # ... other services ...
```

#### 4. Enable HTTPS

**Using Let's Encrypt with Certbot:**
```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# Get certificate
sudo certbot certonly --standalone -d yourdomain.com

# Set auto-renewal
sudo systemctl enable certbot.timer
```

#### 5. Firewall Configuration

**Expose only necessary ports:**
```bash
# Allow web traffic
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Do NOT expose database or search ports
# Database access should be internal only
sudo ufw deny 5432/tcp
sudo ufw deny 7700/tcp
```

### Production docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: xr_edu_postgres
    restart: always
    environment:
      POSTGRES_DB: linkwarden
      POSTGRES_USER: linkwarden_user
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - xr_edu_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U linkwarden_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  meilisearch:
    image: getmeili/meilisearch:v1.12.8
    container_name: xr_edu_meilisearch
    restart: always
    environment:
      MEILI_MASTER_KEY: ${MEILI_MASTER_KEY}
      MEILI_ENV: production
    volumes:
      - meilisearch_data:/meili_data
    networks:
      - xr_edu_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:7700/health"]
      interval: 10s
      timeout: 5s
      retries: 5

  linkwarden:
    image: ghcr.io/linkwarden/linkwarden:latest
    container_name: xr_edu_linkwarden
    restart: always
    env_file: .env
    environment:
      DATABASE_URL: postgresql://linkwarden_user:${POSTGRES_PASSWORD}@postgres:5432/linkwarden
    depends_on:
      postgres:
        condition: service_healthy
      meilisearch:
        condition: service_healthy
    volumes:
      - linkwarden_data:/data/data
    networks:
      - xr_edu_network
    expose:
      - "3000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/v1/auth/status"]
      interval: 30s
      timeout: 10s
      retries: 3

  nginx:
    image: nginx:alpine
    container_name: xr_edu_nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certs:/etc/letsencrypt:ro
      - ./nginx_logs:/var/log/nginx
    depends_on:
      - linkwarden
    networks:
      - xr_edu_network

volumes:
  postgres_data:
    driver: local
  meilisearch_data:
    driver: local
  linkwarden_data:
    driver: local

networks:
  xr_edu_network:
    driver: bridge
```

### Deployment Commands

```bash
# Start production services
docker-compose -f docker-compose.yml up -d

# View logs
docker-compose logs -f linkwarden

# Update image
docker-compose pull
docker-compose up -d

# Backup database
docker-compose exec postgres pg_dump -U linkwarden_user linkwarden > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore database
docker-compose exec -T postgres psql -U linkwarden_user -d linkwarden < backup_file.sql
```

## Monitoring & Maintenance

### Health Checks

```bash
# Check service status
docker-compose ps

# Monitor logs
docker-compose logs --tail=100 -f

# Check specific service
docker-compose logs linkwarden
```

### Performance Monitoring

```bash
# View resource usage
docker stats

# Check disk space
docker exec xr_edu_postgres du -sh /var/lib/postgresql/data
```

### Database Maintenance

```bash
# Connect to database
docker-compose exec postgres psql -U linkwarden_user -d linkwarden

# Inside psql:
-- Vacuum and optimize
VACUUM ANALYZE;

-- Check database size
SELECT pg_size_pretty(pg_database_size('linkwarden'));

-- List largest tables
SELECT schemaname, tablename, 
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) 
FROM pg_tables 
WHERE schemaname != 'pg_catalog' 
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Backup Strategy

### Automated Backups

**Create `backup.sh`:**
```bash
#!/bin/bash

BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
DB_FILE="$BACKUP_DIR/linkwarden_$DATE.sql"
DATA_FILE="$BACKUP_DIR/linkwarden_data_$DATE.tar.gz"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup database
docker-compose exec -T postgres pg_dump -U linkwarden_user linkwarden > $DB_FILE
gzip $DB_FILE

# Backup application data
tar -czf $DATA_FILE ./data

# Keep last 30 days of backups
find $BACKUP_DIR -type f -mtime +30 -delete

echo "Backup completed: $(date)"
```

**Schedule with cron (Linux/Mac):**
```cron
# Backup daily at 2 AM
0 2 * * * cd /path/to/linkwarden && bash backup.sh
```

**Schedule with Windows Task Scheduler:**
1. Create batch file `backup.bat`
2. Schedule task to run daily
3. Execute: `docker-compose exec -T postgres pg_dump ...`

## Scaling Considerations

### Handling Increased Load

#### 1. Increase Resources
```yaml
services:
  postgres:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 8G
  linkwarden:
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
  meilisearch:
    deploy:
      resources:
        limits:
          memory: 4G
```

#### 2. Database Connection Pool
```env
DATABASE_POOL_SIZE=20
DATABASE_STATEMENT_CACHE_SIZE=20
```

#### 3. Multiple Web Containers
```yaml
services:
  linkwarden:
    deploy:
      replicas: 2
```

Add load balancer (nginx) to distribute traffic.

## Disaster Recovery

### Disaster Scenarios

#### Scenario 1: Database Corruption
```bash
# Restore from latest backup
docker-compose down
docker-compose up -d postgres
docker-compose exec -T postgres psql -U linkwarden_user -d linkwarden < latest_backup.sql
docker-compose up -d
```

#### Scenario 2: Complete Data Loss
```bash
# Restore everything
docker-compose down -v  # WARNING: Deletes data
docker-compose up -d
docker-compose exec -T postgres psql -U linkwarden_user -d linkwarden < latest_backup.sql
```

#### Scenario 3: Container Corruption
```bash
# Pull latest image
docker-compose pull

# Recreate containers
docker-compose up -d --force-recreate
```

## Environment Variables Reference

### Required for Production

| Variable | Description | Example |
|----------|-------------|---------|
| `NEXTAUTH_URL` | Auth callback URL | `https://yourdomain.com/api/v1/auth` |
| `NEXTAUTH_SECRET` | Secret key for auth | `base64-encoded-string` |
| `DATABASE_URL` | PostgreSQL connection | `postgresql://user:pass@postgres:5432/linkwarden` |
| `POSTGRES_PASSWORD` | Database password | `secure-password` |

### Recommended for Production

| Variable | Default | Recommendation |
|----------|---------|-----------------|
| `STORAGE_FOLDER` | `./storage` | `/data/data` |
| `NEXT_PUBLIC_DEMO` | `false` | `false` |
| `NEXT_PUBLIC_DISABLE_REGISTRATION` | `false` | `true` (after initial setup) |
| `MAX_WORKERS` | `4` | Adjust to CPU count |

## Support & Resources

- **Docker Compose Documentation:** https://docs.docker.com/compose
- **Linkwarden Docs:** https://docs.linkwarden.app
- **Container Registry:** https://ghcr.io/linkwarden/linkwarden

---

**Last Updated:** March 10, 2026
