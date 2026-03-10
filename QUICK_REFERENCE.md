# XR Educational Bookmark Platform - Quick Reference

## 🚀 Quick Start Commands

### Docker Deployment (Recommended)

```bash
# Start services
cd c:\Users\nunoo\Documents\GitHub\XR_edu\linkwarden
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Restart services
docker-compose restart
```

### Access Points

| Service | URL | Purpose |
|---------|-----|---------|
| Linkwarden Web | http://localhost:3000 | Main application |
| PostgreSQL | localhost:5432 | Database (internal) |
| MeiliSearch | http://localhost:7700 | Search engine API |

## 📝 Essential Configuration

### .env File Location
`c:\Users\nunoo\Documents\GitHub\XR_edu\linkwarden\.env`

### Critical Settings

```env
# Always Set These
NEXTAUTH_URL=http://localhost:3000/api/v1/auth
NEXTAUTH_SECRET=your-random-secret (generate: openssl rand -base64 32)
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/linkwarden
POSTGRES_PASSWORD=postgres
```

### Recommended Additional Settings

```env
# Storage
STORAGE_FOLDER=./storage

# Search
SEARCH_FILTER_LIMIT=1000

# Pagination
PAGINATION_TAKE_COUNT=50

# Instance
NEXT_PUBLIC_DEMO=false
NEXT_PUBLIC_DISABLE_REGISTRATION=false
NEXT_PUBLIC_ADMIN=admin@linkwarden.app
```

## 🗄️ Database Backup & Restore

### Backup Database

```bash
docker-compose exec postgres pg_dump -U postgres linkwarden > backup_$(date +%Y%m%d).sql
```

### Restore Database

```bash
docker-compose exec -T postgres psql -U postgres < backup_YYYYMMDD.sql
```

### Access Database Directly

```bash
docker-compose exec postgres psql -U postgres -d linkwarden
```

## 🔗 API Quick Reference

### Authentication
```bash
# Get API token
curl -X POST http://localhost:3000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@linkwarden.app","password":"your_password"}'
```

### Collections API
```bash
# List collections
curl http://localhost:3000/api/v1/collections

# Create collection
curl -X POST http://localhost:3000/api/v1/collections \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"My Collection","description":"Description"}'
```

### Links API
```bash
# Add link
curl -X POST http://localhost:3000/api/v1/links \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com","collection":"1","description":"Link description"}'

# Search links
curl "http://localhost:3000/api/v1/links/search?q=xr&collection=1"

# Get link details
curl http://localhost:3000/api/v1/links/LINK_ID
```

## 🏷️ XR Content Tagging Quick Guide

### Platform Tags
- `#quest-3`, `#quest-pro` - Meta platforms
- `#apple-vision-pro` - Apple spatial computing
- `#webxr` - Browser-based
- `#pico` - Bytedance platform

### Subject Tags
- `#stem`, `#science`, `#physics`, `#biology`, `#chemistry`
- `#mathematics`, `#geometry`, `#stem`
- `#languages`, `#history`, `#geography`, `#arts`
- `#vocational`, `#professional-development`

### Pedagogical Tags
- `#interactive`, `#simulation`, `#game-based`, `#immersive`
- `#collaborative`, `#individual-learning`, `#hands-on`
- `#constructivist`, `#project-based`, `#problem-based`

### Age/Grade Tags
- `#elementary`, `#middle-school`, `#high-school`, `#university`
- `#ages-5-8`, `#ages-9-12`, `#ages-13-18`, `#ages-18+`

### Accessibility Tags
- `#wheelchair-accessible`, `#hand-tracking`, `#seated-vr`
- `#colorblind-friendly`, `#high-contrast`, `#captions`
- `#haptic-feedback`, `#voice-input`, `#eye-tracking`

## 🛠️ Common Troubleshooting

### Services Won't Start

```bash
# Check what's using ports
netstat -ano | findstr :3000
netstat -ano | findstr :5432
netstat -ano | findstr :7700

# Force stop existing process (if needed)
taskkill /PID <PID> /F

# Restart Docker Desktop
# (Restart application or computer)
```

### Database Connection Error

```bash
# Verify services are running
docker-compose ps

# Check logs
docker-compose logs postgres

# Restart PostgreSQL
docker-compose restart postgres

# Recreate database
docker-compose down -v
docker-compose up -d
```

### UI Not Loading

```bash
# Check web app logs
docker-compose logs web

# Restart web service
docker-compose restart web

# Clear browser cache and hard reload (Ctrl+Shift+R)
```

### Search Not Working

```bash
# Check MeiliSearch status
curl http://localhost:7700/health

# Restart search service
docker-compose restart meilisearch

# Re-index content (in app settings)
```

## 📱 Mobile Access

### Local Network Access
```
http://your-computer-ip:3000
# Example: http://192.168.1.100:3000
```

### Remote Access Setup
1. Configure reverse proxy (nginx, Apache)
2. Set up HTTPS certificate
3. Use dynamic DNS for domain
4. Update NEXTAUTH_URL in .env

## 🔐 Security Checklist

- [ ] Change `NEXTAUTH_SECRET` (run: `openssl rand -base64 32`)
- [ ] Change `POSTGRES_PASSWORD`
- [ ] Ensure non-default admin credentials
- [ ] Enable HTTPS for external access
- [ ] Restrict database firewall rules
- [ ] Set up regular backups
- [ ] Monitor container resource usage
- [ ] Keep Docker images updated

## 📊 Performance Optimization

### Docker Resource Limits

Edit `docker-compose.yml`:

```yaml
services:
  linkwarden:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          memory: 1G
  postgres:
    deploy:
      resources:
        limits:
          memory: 2G
```

### Database Optimization

```sql
-- Check database size
SELECT pg_size_pretty(pg_database_size('linkwarden'));

-- Vacuum and analyze
VACUUM ANALYZE;

-- Check table sizes
SELECT schemaname, tablename, 
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) 
FROM pg_tables ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## 📖 Useful File Locations

### Configuration Files
- **Docker Compose:** `linkwarden/docker-compose.yml`
- **Environment:** `linkwarden/.env`
- **Dockerfile:** `linkwarden/Dockerfile`

### Documentation (This Project)
- **Setup Guide:** `SETUP_GUIDE.md`
- **Customization:** `CUSTOMIZATION_GUIDE.md`
- **Collection Templates:** `COLLECTION_TEMPLATES.md`
- **README:** `README.md`

### Data Directories
- **Database Data:** `linkwarden/pgdata/`
- **Application Data:** `linkwarden/data/`
- **Search Index:** `linkwarden/meili_data/`

## 🆘 Getting Help

### Documentation
- [Linkwarden Official Docs](https://docs.linkwarden.app)
- [API Documentation](https://docs.linkwarden.app/self-hosting/api)
- [Docker Compose Docs](https://docs.docker.com/compose)

### Community Support
- [Discord Community](https://discord.com/invite/CtuYV47nuJ)
- [GitHub Issues](https://github.com/linkwarden/linkwarden/issues)
- [GitHub Discussions](https://github.com/linkwarden/linkwarden/discussions)

### Local Development
- Check Docker logs: `docker-compose logs -f`
- Test API endpoints: use Postman or curl
- Debug UI: browser DevTools (F12)

## ⚡ Next Steps

1. ✅ Start Docker services: `docker-compose up -d`
2. ✅ Access http://localhost:3000
3. ✅ Create first collection
4. ✅ Add test bookmarks
5. ✅ Configure team members
6. ✅ Customize for XR content
7. ✅ Set up backup strategy
8. ✅ Deploy to production

## 📞 Contact & Support

- **Framework:** [Linkwarden](https://linkwarden.app)
- **Project:** XR Educational Bookmark Platform
- **Last Updated:** March 10, 2026

---

For detailed setup instructions, see [SETUP_GUIDE.md](./SETUP_GUIDE.md)
For customization details, see [CUSTOMIZATION_GUIDE.md](./CUSTOMIZATION_GUIDE.md)
