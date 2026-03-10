# XR Educational Content Customization Guide

## Overview

This guide explains how to customize Linkwarden specifically for XR educational content management.

## 1. Database Schema Extensions

### Adding XR-Specific Metadata Fields

The Prisma schema is located in `packages/prisma/schema.prisma`. To add XR-specific fields:

```prisma
// Add to the Link model
model Link {
  // ... existing fields ...
  
  // XR-specific fields
  xrPlatform      String?        // Meta Quest, Apple Vision Pro, WebXR, etc.
  xrCategory      String?        // Application, Game, Educational Tool, etc.
  pedagogicalGoal String?        // Learning objectives
  targetAgeMin    Int?           // Minimum age/grade
  targetAgeMax    Int?           // Maximum age/grade
  duration        Int?           // Duration in minutes
  costType        String?        // Free, Freemium, Paid
  accessibilityNotes String?    // Accessibility features
  requiredHardware String?       // Hardware requirements
}
```

### Implementation Steps

1. Modify `packages/prisma/schema.prisma`
2. Run migration: `yarn prisma:dev`
3. Generate new Prisma client: `yarn prisma:generate`

## 2. Custom Collections Setup

### Recommended Collection Structure

```
XR Educational Resources/
├── VR Applications
│   ├── Science & Physics
│   ├── Language Learning
│   ├── History & Social Studies
│   └── Mathematics
├── AR Experiences
│   ├── Mobile AR Apps
│   ├── Web-based AR
│   └── Hardware-based AR
├── Educational Games
│   ├── Serious Games
│   ├── Simulation Games
│   └── Puzzle & Strategy
├── Learning Videos
│   ├── VR Tutorials
│   ├── Platform Guides
│   └── Pedagogy Resources
└── Research & Documentation
    ├── Academic Papers
    ├── Implementation Guides
    └── Safety & Ethics
```

## 3. Enhanced Tagging System

### Recommended Tags

#### Platform Tags
- `quest-2`, `quest-3`, `pico`, `vive`, `psvr`, `apple-vision-pro`, `webxr`

#### Subject Area Tags
- `stem`, `science`, `biology`, `physics`, `chemistry`
- `mathematics`, `geometry`, `algebra`, `calculus`
- `languages`, `history`, `geography`, `arts`, `music`
- `vocational`, `professional-development`

#### Learning Type Tags
- `interactive`, `simulation`, `game-based`, `immersive`
- `collaborative`, `individual-learning`, `hands-on`

#### Age/Grade Tags
- `elementary`, `middle-school`, `high-school`, `university`
- `age-5-8`, `age-9-12`, `age-13-18`, `age-18+`

#### Technical Tags
- `free`, `freemium`, `paid`, `open-source`
- `requires-motion-controllers`, `requires-hand-tracking`
- `wheelchair-accessible`, `hearing-accessible`, `vision-accessible`

#### Pedagogical Tags
- `constructivist`, `behaviorist`, `connectivist`
- `project-based`, `problem-based`, `inquiry-based`
- `active-learning`, `experiential-learning`

## 4. UI Customization

### Branding Changes

**Web App Directory:** `apps/web`

Key files for customization:

```
apps/web/
├── public/
│   └── favicon.ico          # Change app icon
├── src/
│   ├── components/
│   │   └── Brand/           # Logo and branding
│   ├── styles/
│   │   └── globals.css      # Color scheme
│   └── pages/
│       ├── _app.tsx         # App layout
│       └── _document.tsx    # Document structure
```

### Color Scheme for XR Education

Recommended colors:
- **Primary:** `#2563EB` (Blue - for trust/education)
- **Secondary:** `#8B5CF6` (Purple - for XR/tech)
- **Accent:** `#10B981` (Green - for interactive/active)
- **Warning:** `#F59E0B` (Amber - for important info)

### Theme Configuration

Modify in `apps/web/tailwind.config.ts`:

```typescript
module.exports = {
  theme: {
    extend: {
      colors: {
        xr: {
          50: '#f0f9ff',
          500: '#8B5CF6', // XR Purple
          900: '#5B21B6',
        },
        education: {
          50: '#eff6ff',
          500: '#2563EB', // Education Blue
          900: '#1e3a8a',
        }
      }
    }
  }
}
```

## 5. Custom Navigation Menu

To add XR-specific navigation, modify the menu in:
`apps/web/src/components/Layout/Navigation.tsx`

```typescript
const xrMenuItems = [
  { label: 'VR Applications', href: '/collections/vr-applications' },
  { label: 'AR Experiences', href: '/collections/ar-experiences' },
  { label: 'Educational Games', href: '/collections/games' },
  { label: 'Learning Videos', href: '/collections/videos' },
  { label: 'Research', href: '/collections/research' }
];
```

## 6. API Extensions

### Custom Endpoints for XR Content

Add to `apps/web/src/pages/api/v1/`:

```typescript
// pages/api/v1/links/xr-stats.ts
export default async function handler(req, res) {
  // Get statistics on XR content
  const stats = {
    totalXrLinks: 0,
    byPlatform: {},
    bySubject: {},
    byAgeGroup: {}
  };
  res.status(200).json(stats);
}
```

## 7. Import/Export Features

### Bulk Import from CSV

Create a CSV template for XR resources:

```csv
title,url,description,xrPlatform,xrCategory,targetAgeMin,targetAgeMax,pedagogicalGoal,tags
"3D Anatomy VR","https://example.com/anatomy","Interactive anatomy","quest-3","Application","13","18","Medical education","science,biology,interactive"
```

### Export Reports

Generate collection reports with XR-specific metrics:
- Platform distribution
- Subject area breakdown
- Age group targeting
- Accessibility compliance

## 8. Security & Privacy for Educational Use

### FERPA Compliance (US)
- Enable user data privacy: `NEXT_PUBLIC_DISABLE_REGISTRATION=false`
- Implement access logs for audit trail
- Regular data backups

### GDPR Compliance (EU)
- Implement data export functionality
- Add "right to be forgotten" feature
- Privacy policy documentation

### Educational Best Practices
- Content moderation workflows
- Age-appropriate filtering
- Teacher/student role separation
- Parental consent workflows

## 9. Integration Examples

### Browser Extension Integration

The official Linkwarden browser extension enables easy bookmarking:
- Save educational XR resources while browsing
- Add metadata on the fly
- Automatic categorization

**Installation:**
- Chrome/Brave: [Extension Link](https://github.com/linkwarden/browser-extension)

### LMS Integration

Potential integrations with learning management systems:
- Moodle
- Canvas
- Blackboard
- Google Classroom

### Webhook Integration Example

```bash
# Webhook to notify when new XR content is added
POST /webhooks/xr-content-added
Content-Type: application/json

{
  "linkId": "123",
  "title": "New XR App",
  "xrPlatform": "quest-3",
  "xrCategory": "Educational Tool",
  "tags": ["science", "interactive"]
}
```

## 10. Deployment Customization

### Environment Variables for XR Edition

```env
# XR-Specific Settings
NEXT_PUBLIC_XR_PLATFORM_FILTER=true
NEXT_PUBLIC_SHOW_PEDAGOGICAL_METADATA=true
NEXT_PUBLIC_ENABLE_AGE_FILTERING=true
NEXT_PUBLIC_COLLECTION_TEMPLATES=true

# Default Collections
DEFAULT_COLLECTIONS=vr-apps,ar-experiences,games,videos

# Search Preferences
DEFAULT_XR_TAGS=interactive,immersive,educational
```

### Docker Production Build

Modify `Dockerfile`:

```dockerfile
# ... existing configuration ...

ENV NEXT_PUBLIC_APP_NAME="XR Educational Platform"
ENV NEXT_PUBLIC_BRANDING_COLOR="#8B5CF6"
ENV NEXT_PUBLIC_ORGANIZATION="Your Institution"

# Multi-stage build for optimization
FROM node:19-alpine AS builder
# ... build steps ...
```

## 11. Analytics & Reporting

### Usage Metrics to Track

- Most accessed XR platforms
- Popular pedagogical approaches
- Subject area distribution
- Age group engagement
- Resource engagement patterns

### Dashboard Widget Example

Create custom reports showing:
```
📊 XR Content Dashboard
├── Resources by Platform (pie chart)
├── Content Distribution by Subject (bar chart)
├── User Engagement Metrics (timeline)
└── Accessibility Compliance Score
```

## 12. Community & Feedback

### User Feedback Integration

Add feedback collection using:
- In-app surveys
- Community Discord channel
- GitHub discussions
- Feature request voting

### Contribution Guidelines

Enable community participation:
- Curation guidelines
- Content review process
- Quality standards
- Moderation policies

## Getting Help

- **Linkwarden Docs:** https://docs.linkwarden.app
- **Customization Support:** [GitHub Issues](https://github.com/linkwarden/linkwarden/issues)
- **Community Discord:** https://discord.com/invite/CtuYV47nuJ

## Next Steps

1. ✅ Review database schema for XR fields
2. Set up collection hierarchy
3. Implement custom tags system
4. Customize branding
5. Deploy to production
6. Gather user feedback
7. Iterate and improve

---

**Ready to customize!** Choose your priority customizations and implement them step by step.
