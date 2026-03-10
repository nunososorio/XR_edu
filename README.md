# 🌐 XR Educational Bookmark Platform

A collaborative bookmark management system for collecting, organizing, and sharing XR applications, educational videos, games, and learning experiences with pedagogical value.

Built on [**Linkwarden**](https://linkwarden.app) - an open-source, self-hosted bookmark manager.

## 🌍 Live Documentation

**Website:** https://nunososorio.github.io/XR_EDU

Your complete documentation and landing page is published on GitHub Pages with automatic updates on every push!

## 📋 Table of Contents

- [Features](#-features)
- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [Customization](#-customization)
- [Development](#-development)
- [Deployment](#-deployment)
- [Contributing](#-contributing)
- [License](#-license)

## ✨ Features

### Core Platform Capabilities

- 📸 **Auto-Capture** - Screenshots, PDFs, and full-page HTML of each resource
- 📖 **Reader View** - Distraction-free reading with highlighting and annotations
- 🏛️ **Archive Integration** - Send resources to Wayback Machine for preservation
- ✨ **Smart Tagging** - AI-assisted categorization (optional with Ollama)
- 📂 **Collections** - Hierarchical organization with sub-collections
- 👥 **Collaboration** - Invite members with customizable permissions
- 🌐 **Public Sharing** - Share collections and enable RSS feeds
- 🔍 **Full-Text Search** - Powered by MeiliSearch for instant results
- 📱 **Mobile Support** - Native iOS and Android apps available
- 🌓 **Dark/Light Mode** - Eye-friendly interface options

### XR Educational Enhancements

- 🎮 **XR Platform Tagging** - Meta Quest, Apple Vision Pro, WebXR, Pico, etc.
- 🎯 **Pedagogical Metadata** - Learning objectives, age targeting, duration, cost
- 📚 **Subject Area Filtering** - STEM, Languages, History, Arts, Vocational, etc.
- 🏆 **Learning Type Classification** - Interactive, Simulation, Game-based, Immersive
- ♿ **Accessibility Tracking** - Hardware requirements, accessibility features
- 📊 **Educational Analytics** - Usage metrics and engagement reporting
- 🤝 **Team Collaboration** - Share curated collections with educators and students

## 🚀 Quick Start

### Requirements

- Docker Desktop (recommended)
- or: Node.js v19+, PostgreSQL 16+, Yarn 4+

### Option 1: Docker Compose (Recommended)

```bash
# Navigate to project
cd linkwarden

# Start services
docker-compose up -d

# Access at http://localhost:3000
```

**Services Started:**
- Linkwarden Web: http://localhost:3000
- PostgreSQL Database: localhost:5432
- MeiliSearch: localhost:7700

### Option 2: Local Development

```bash
# Install dependencies
yarn install

# Generate Prisma client
yarn prisma:generate

# Run development servers
yarn concurrently:dev

# Access at http://localhost:3000
```

## 📁 Project Structure

```
XR_edu/
├── linkwarden/                  # Cloned Linkwarden repository
│   ├── apps/
│   │   ├── web/                 # Next.js web application
│   │   └── worker/              # Background job processor
│   ├── packages/
│   │   ├── prisma/              # Database schema & migrations
│   │   └── utils/               # Shared utilities
│   ├── docker-compose.yml       # Docker services configuration
│   ├── Dockerfile               # Application container
│   └── .env                     # Environment configuration
│
├── SETUP_GUIDE.md               # Installation & deployment guide
├── CUSTOMIZATION_GUIDE.md       # XR-specific customization docs
└── README.md                    # This file
```

## 🎨 Customization

See [CUSTOMIZATION_GUIDE.md](./CUSTOMIZATION_GUIDE.md) for detailed customization instructions:

### 1. Database Schema Extensions
Add XR-specific fields to track platform, pedagogical goals, age targeting, etc.

### 2. Collection Templates
Pre-configured hierarchies for VR Apps, AR Experiences, Games, and Videos

### 3. Enhanced Tagging
Platform tags, subject areas, learning types, accessibility features

### 4. Branding & UI
Customize colors, logos, and navigation for your institution

### 5. API Extensions
Build custom endpoints for XR content analytics and reporting

### 6. Integrations
Connect with LMS, webhooks, and community platforms

## 🔧 Development

### Local Setup

```bash
# Clone base repository (already done)
git clone https://github.com/linkwarden/linkwarden.git

# Install dependencies
yarn install

# Configure environment
cp .env.sample .env
# Edit .env with your settings

# Run database migrations
yarn prisma:deploy

# Start development servers
yarn concurrently:dev
```

### Useful Commands

```bash
# Database
yarn prisma:generate      # Generate Prisma client
yarn prisma:dev          # Open Prisma Studio
yarn prisma:deploy       # Run migrations
yarn prisma:studio       # Database UI

# Build
yarn web:build           # Build web app for production
yarn web:start           # Start production server

# Development
yarn web:dev             # Start web dev server
yarn worker:dev          # Start worker dev server
yarn format              # Format code
```

### Making Code Changes

#### Modifying the Database Schema

1. Edit `packages/prisma/schema.prisma`
2. Create migration: `yarn prisma:dev`
3. Generate client: `yarn prisma:generate`
4. Build and test

#### Customizing the Web UI

1. Modify files in `apps/web/src/`
2. Changes reflect instantly in dev mode
3. Components: `components/`, Pages: `pages/`, Styles: `styles/`

#### Adding API Endpoints

1. Create file in `apps/web/src/pages/api/v1/`
2. Export default handler function
3. Test with curl or Postman

## 🌍 Deployment

### Production Docker Deployment

See [SETUP_GUIDE.md](./SETUP_GUIDE.md#security-recommendations) for security setup.

```bash
# Build production image
docker-compose build

# Start with production settings
docker-compose up -d

# Verify services
docker-compose ps
```

### Cloud Deployment Options

- **Vercel** - Official Next.js hosting (frontend only)
- **Railway/Render** - Full-stack deployment
- **DigitalOcean/Linode** - VPS with Docker
- **AWS/Azure/GCP** - Enterprise deployment
- **Docker Hub** - Push custom image

### Backup Strategy

```bash
# Backup database
docker-compose exec postgres pg_dump -U postgres linkwarden > backup.sql

# Restore from backup
docker-compose exec -T postgres psql -U postgres < backup.sql

# Backup storage folder
tar -czf storage_backup.tar.gz ./data
```

## 📚 Documentation

- [Setup Guide](./SETUP_GUIDE.md) - Installation and configuration
- [Customization Guide](./CUSTOMIZATION_GUIDE.md) - XR-specific enhancements
- [Linkwarden Docs](https://docs.linkwarden.app) - Official documentation
- [API Documentation](https://docs.linkwarden.app/self-hosting/api) - REST API reference

## 🤝 Contributing

### How to Contribute

1. Fork the base [Linkwarden repository](https://github.com/linkwarden/linkwarden)
2. Create feature branch: `git checkout -b feature/xr-enhancement`
3. Make changes and test locally
4. Submit pull request with clear description

### XR-Specific Contributions Welcome

- Additional XR platform support
- New pedagogical metadata fields
- Educational integrations (Moodle, Canvas, etc.)
- Accessibility improvements
- Translation and localization

## 🔐 Security & Compliance

### Built-in Security

- Self-hosted data ownership
- No third-party analytics
- HTTPS support (via reverse proxy)
- Database encryption compatible
- API key authentication
- Role-based access control

### Educational Compliance

- FERPA-compatible (US - Family Educational Rights and Privacy Act)
- GDPR-ready (EU - General Data Protection Regulation)
- COPPA considerations (US - Children's Online Privacy Protection)
- Content moderation workflows
- User activity audit trails

### Securing Your Installation

1. Change default passwords
2. Generate strong `NEXTAUTH_SECRET`
3. Enable HTTPS with reverse proxy
4. Regular database backups
5. Monitor system resources
6. Keep dependencies updated

## 📊 Architecture

### Technology Stack

- **Frontend:** Next.js 14, React, TypeScript, Tailwind CSS
- **Backend:** Node.js, Express integration
- **Database:** PostgreSQL 16
- **Search:** MeiliSearch
- **Authentication:** NextAuth.js
- **Deployment:** Docker, Docker Compose

### Key Components

- **Web App** (`apps/web`) - React/Next.js SPA
- **Worker** (`apps/worker`) - Background job processor
- **Prisma** (`packages/prisma`) - ORM and migrations
- **PostgreSQL** - Primary data store
- **MeiliSearch** - Full-text search engine

## 📈 Roadmap

### Phase 1: Core Setup ✅
- [x] Clone Linkwarden repository
- [x] Configure Docker environment
- [x] Create documentation
- [x] Set up project structure

### Phase 2: Customization (In Progress)
- [ ] Add XR-specific database fields
- [ ] Implement custom tagging system
- [ ] Create collection templates
- [ ] Build pedagogical metadata system

### Phase 3: Integration
- [ ] LMS connectors (Moodle, Canvas)
- [ ] Educational analytics dashboard
- [ ] Content moderation workflows
- [ ] Mobile app customization

### Phase 4: Community
- [ ] Community content curation
- [ ] Educator resource sharing
- [ ] Student collaboration features
- [ ] Accessibility improvements

## 📞 Support

### Getting Help

- **GitHub Issues:** [Linkwarden Issues](https://github.com/linkwarden/linkwarden/issues)
- **Discord Community:** [Linkwarden Discord](https://discord.com/invite/CtuYV47nuJ)
- **Documentation:** [docs.linkwarden.app](https://docs.linkwarden.app)
- **Demos:** [demo.linkwarden.app](https://demo.linkwarden.app)

### Reporting Issues

When reporting issues, include:
- Error messages and logs
- Steps to reproduce
- Environment details (Docker/Local, OS, version)
- Expected vs actual behavior

## 📄 License

This project is licensed under the **GNU Affero General Public License v3.0** (AGPL-3.0), same as Linkwarden.

This means:
- ✅ Free to use and modify
- ✅ Source code must remain open
- ✅ Modifications must be shared with users
- ✅ Can be used commercially
- ⚠️ Must provide source code access to all users

See [LICENSE.md](./linkwarden/LICENSE.md) for full details.

## 🙏 Acknowledgments

- **Linkwarden Team** - Excellent open-source bookmark platform
- **Linkwarden Community** - Vibrant and supportive user base
- **XR Educators** - Inspiration for pedagogical features
- **Open Source Community** - All contributing libraries and tools

## 📝 Changelog

### v1.0 (Current)
- Initial XR Educational Platform setup
- Based on Linkwarden 2.10.0
- Complete documentation
- Docker deployment ready
- Customization guide

### Future Versions
- XR-specific database schema
- Enhanced UI customizers
- Educational integrations
- Analytics dashboard

---

## 🚀 Get Started Now

1. **Read** the [Setup Guide](./SETUP_GUIDE.md)
2. **Run** Docker Compose
3. **Access** http://localhost:3000
4. **Create** first collection
5. **Customize** for your needs

**Questions?** Check the documentation or join the [community](https://discord.com/invite/CtuYV47nuJ)!

---

**Last Updated:** March 10, 2026  
**Maintained By:** Your Institution Name  
**Status:** Active Development 🔄
