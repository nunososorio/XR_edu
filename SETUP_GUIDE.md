# XR Educational Bookmark Platform - Setup Guide

Based on **Linkwarden** - A self-hosted, collaborative bookmark management platform

## Overview

This XR Educational Bookmark Platform is built on Linkwarden, customized for collecting, organizing, and sharing:
- XR Applications & Tools
- Educational Videos  
- VR/AR Games and Experiences
- Learning Resources with pedagogical value

## Quick Start with Docker Compose

### Prerequisites

- Docker Desktop installed
- Windows 10/11 or later
- ~5GB disk space for database and storage

### Installation Steps

1. **Navigate to the project directory:**
```bash
cd c:\Users\nunoo\Documents\GitHub\XR_edu\linkwarden
```

2. **Verify the .env file:**
The file has been created with basic configuration. Key settings:
- `NEXTAUTH_URL=http://localhost:3000/api/v1/auth`
- `DATABASE_URL=postgresql://postgres:postgres@localhost:5432/linkwarden`
- `POSTGRES_PASSWORD=postgres`
- `STORAGE_FOLDER=./storage`

3. **Start the services:**
```bash
docker-compose up -d
```

This starts:
- **Linkwarden Web** on http://localhost:3000
- **PostgreSQL Database**
- **MeiliSearch** for full-text search capabilities

4. **Access the application:**
Open browser: http://localhost:3000

### Initial Login

Default admin credentials (can be customized in .env):
- **Email:** admin@linkwarden.app
- **Password:** (Create your own during first login)

## Features Available

### Core Bookmark Management
- ✅ Capture screenshots, PDFs, and HTML of webpages
- ✅ Reader view with text highlighting and annotations
- ✅ Full-text search with MeiliSearch integration
- ✅ Collections and sub-collections for organization
- ✅ Tags and categories (AI-assisted optional)
- ✅ Archive links to Wayback Machine

### Collaboration
- ✅ Invite team members with customizable permissions
- ✅ Share collections publicly
- ✅ RSS feed subscriptions
- ✅ Bulk actions and import

### XR-Specific Customizations

The platform can be customized to include:

1. **Custom Collections for XR**
   - VR Applications
   - AR Experiences
   - Educational Games
   - Pedagogical Resources

2. **Enhanced Metadata**
   - XR Platform type (Meta Quest, Apple Vision Pro, WebXR, etc.)
   - Learning objectives and pedagogical approach
   - Age/Grade level targeting
   - Time duration
   - Cost (Free, Freemium, Paid)

3. **Enhanced Tagging System**
   - Subject areas (STEM, Languages, History, etc.)
   - Learning type (Interactive, Simulation, Game-based)
   - Hardware requirements
   - Accessibility features

## Advanced Configuration

### Email Setup (Optional)
```env
NEXT_PUBLIC_EMAIL_PROVIDER=smtp
EMAIL_FROM=noreply@yourdomain.com
EMAIL_SERVER=smtp://user:password@host:port
BASE_URL=https://yourdomain.com
```

### AI Tagging (Optional - Local)
```env
NEXT_PUBLIC_OLLAMA_ENDPOINT_URL=http://localhost:11434
OLLAMA_MODEL=llama2
```

### Search Settings
```env
SEARCH_FILTER_LIMIT=1000
TEXT_CONTENT_LIMIT=500000
```

## File Structure

```
linkwarden/
├── apps/
│   ├── web/              # Next.js web application
│   └── worker/           # Background job processor
├── packages/
│   ├── prisma/           # Database schema
│   └── utils/            # Shared utilities
├── docker-compose.yml    # Docker services configuration
├── .env                  # Environment variables
└── Dockerfile            # Application container setup
```

## Development & Customization

For local development without Docker:

### Prerequisites
- Node.js v19+
- PostgreSQL 16+
- Yarn 4+

### Steps
```bash
# Install dependencies
yarn install

# Generate Prisma client
yarn prisma:generate

# Run development servers
yarn concurrently:dev

# Build for production
yarn web:build
yarn worker:start
```

## Database Management

### Access PostgreSQL
```bash
docker-compose exec postgres psql -U postgres
```

### View database schema
```bash
docker-compose exec postgres psql -U postgres -d linkwarden -c "\dt"
```

### Backup database
```bash
docker-compose exec postgres pg_dump -U postgres linkwarden > backup.sql
```

## Stopping Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (careful - deletes data)
docker-compose down -v
```

## Security Recommendations

1. **Change default passwords** in .env
2. **Set strong NEXTAUTH_SECRET** (generate with `openssl rand -base64 32`)
3. **Use HTTPS in production** via reverse proxy (nginx, caddy)
4. **Restrict database access** to localhost
5. **Run behind authentication** proxy if exposed publicly
6. **Regular backups** of PostgreSQL data

## Troubleshooting

### Services not starting
```bash
# Check logs
docker-compose logs -f

# Restart services
docker-compose restart
```

### Database connection error
- Ensure PostgreSQL container is running: `docker-compose ps`
- Check DATABASE_URL in .env matches container setup
- Check POSTGRES_PASSWORD matches in both places

### Out of memory
- Increase Docker Desktop memory limit
- Reduce `MAX_WORKERS` in .env

## Next Steps

1. ✅ [Run Docker Compose](#quick-start-with-docker-compose)
2. Access web interface
3. Create first collection for XR resources
4. Invite team members
5. Customize UI/Branding (optional)
6. Set up browser extension for easy link collection
7. Configure email notifications (optional)

## Resources

- [Linkwarden Documentation](https://docs.linkwarden.app)
- [Linkwarden Demo](https://demo.linkwarden.app)
- [GitHub Repository](https://github.com/linkwarden/linkwarden)
- [Discord Community](https://discord.com/invite/CtuYV47nuJ)

## License

Linkwarden is licensed under **GNU Affero General Public License v3.0**

---

**Setup completed!** Your XR Educational Bookmark Platform is ready to use.
