# Developer's Guide - XR Educational Platform Customizations

## Overview

This guide provides step-by-step instructions for developers to customize Linkwarden for XR educational content. It covers database modifications, UI enhancements, and API extensions.

## Table of Contents

1. [Development Environment Setup](#development-environment-setup)
2. [Project Architecture](#project-architecture)
3. [Database Schema Customization](#database-schema-customization)
4. [Frontend Modifications](#frontend-modifications)
5. [API Extensions](#api-extensions)
6. [Styling & Branding](#styling--branding)
7. [Testing & Deployment](#testing--deployment)
8. [Common Customization Scenarios](#common-customization-scenarios)

## Development Environment Setup

### Prerequisites

- Node.js v19+
- PostgreSQL 16+ (running)
- Yarn 4+
- VS Code or preferred editor
- Git

### Local Development Installation

```bash
# Navigate to project
cd c:\Users\nunoo\Documents\GitHub\XR_edu\linkwarden

# Install dependencies
npm install  # Uses yarn workspaces internally via postinstall script

# Generate Prisma client
npx prisma generate --monorepo

# Set up environment for development
cp .env.sample .env

# Configure local database connection (if not using Docker)
# Update NEXT_AUTH_URL, DATABASE_URL, etc. in .env

# Start development servers
npm run concurrently:dev
```

This runs three servers in parallel:
- Web app on http://localhost:3000
- Worker on background
- Database migrations if needed

### Stopping Development

```bash
npm run stop  # or press Ctrl+C in terminal
```

## Project Architecture

### Directory Structure Overview

```
linkwarden/
├── apps/
│   ├── web/                          # Next.js Application
│   │   ├── src/
│   │   │   ├── pages/                # Next.js routes & API routes
│   │   │   │   ├── api/              # REST API endpoints
│   │   │   │   └── ...               # Frontend pages
│   │   │   ├── components/           # React components
│   │   │   │   ├── ui/               # Shared UI components
│   │   │   │   ├── Layout/           # Layout components
│   │   │   │   └── Link/             # Link-related components
│   │   │   ├── hooks/                # Custom React hooks
│   │   │   ├── styles/               # Global styles & Tailwind
│   │   │   ├── utils/                # Utility functions
│   │   │   └── lib/                  # Library configurations
│   │   ├── public/                   # Static assets
│   │   ├── next.config.js            # Next.js configuration
│   │   └── package.json
│   │
│   └── worker/                       # Background Job Processor
│       ├── src/
│       │   ├── jobs/                 # Background jobs
│       │   └── ...
│       └── package.json
│
├── packages/
│   ├── prisma/                       # Database & ORM
│   │   ├── schema.prisma             # Database schema (★ EDIT THIS)
│   │   ├── migrations/               # Database migrations
│   │   └── package.json
│   │
│   └── utils/                        # Shared utilities
│       ├── src/
│       │   ├── helpers/
│       │   └── types/
│       └── package.json
│
├── docker-compose.yml
├── .env
├── package.json
└── yarn.lock
```

### Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Frontend | Next.js 14, React, TypeScript | Web UI |
| Styling | Tailwind CSS | Utility-first CSS |
| Database | PostgreSQL, Prisma ORM | Data persistence |
| Search | MeiliSearch | Full-text search |
| Auth | NextAuth.js | Authentication |
| API | Next.js API Routes | REST endpoints |
| Storage | File system / S3 | Document storage |

## Database Schema Customization

### Understanding Prisma Schema

The database schema is defined in `packages/prisma/schema.prisma`. This is a Prisma DSL file that:
- Defines all models (like Link, Collection, User)
- Specifies relationships between models
- Creates database migrations automatically

### Task: Add XR-Specific Fields to Link Model

#### Step 1: Modify Schema

File: `packages/prisma/schema.prisma`

Find the `Link` model and add XR-specific fields:

```prisma
model Link {
  // ... existing fields ...
  id                Int     @id @default(autoincrement())
  url               String
  title             String?
  description       String?
  collectionId      Int
  userId            Int
  // ... other existing fields ...

  // ★ ADD THESE NEW FIELDS ★
  
  // XR Platform Information
  xrPlatform        String?     // quest-3, quest-pro, apple-vision-pro, webxr, pico, etc.
  xrCategory        String?     // Application, Game, Video, Experience, Tool
  xrDeveloper       String?     // Developer/Company name
  
  // Pedagogical Information
  pedagogicalGoal   String?     // Learning objectives
  pedagogicalType   String?     // Interactive, Simulation, Game-based, Exploration
  subjectArea       String?     // Science, Mathematics, Languages, History, Arts, Vocational
  
  // Targeting Information
  targetAgeMin      Int?        // Minimum age/grade
  targetAgeMax      Int?        // Maximum age/grade
  duration          Int?        // Duration in minutes
  
  // Cost Information
  costType          String?     // Free, Freemium, Paid
  costAmount        Float?      // Price in USD (if paid)
  
  // Accessibility Information
  accessibilityNotes String?    // Accessibility features available
  requiredHardware  String?     // Hardware requirements (controllers, tracking, etc.)
  
  // Additional Metadata
  isVerified        Boolean     @default(false)  // Content verification
  verificationNote  String?     // Why verified (peer review, educator tested, etc.)
  difficultyLevel   String?     // Beginner, Intermediate, Advanced
  
  // ... existing fields continue ...
  
  // Keep existing relationships
  collection        Collection  @relation(fields: [collectionId], references: [id], onDelete: Cascade)
  user              User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([url, collectionId])
  @@index([collectionId])
  @@index([userId])
  @@fulltext([title, description])
}
```

#### Step 2: Create Migration

```bash
cd c:\Users\nunoo\Documents\GitHub\XR_edu\linkwarden

# Generate and create migration
npx prisma migrate dev --name add_xr_fields_to_link

# Or for PostgreSQL specifically
npx prisma migrate dev --name add_xr_fields_to_link
```

This will:
- Create a migration file
- Apply it to your database
- Generate updated Prisma client

#### Step 3: Verify Migration

```bash
# Check that migration was applied
npx prisma migrate status

# Open Prisma Studio to view data
npx prisma studio
```

Access at http://localhost:5555

### Additional Schema Patterns

#### Creating New Models for XR Content

```prisma
model XrPlatform {
  id            Int     @id @default(autoincrement())
  name          String  @unique  // quest-3, apple-vision-pro
  displayName   String  // Meta Quest 3
  icon          String? // Icon URL
  specification String? // Technical specs
  links         Link[]
  
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  
  @@index([name])
}

model PedagogicalFramework {
  id            Int     @id @default(autoincrement())
  name          String  @unique  // Bloom's, Constructivism, etc.
  description   String?
  links         Link[]  // Many-to-many relationship
  
  createdAt     DateTime @default(now())
  
  @@index([name])
}
```

#### Adding Tags/Categories Relationship

```prisma
model Tag {
  id            Int     @id @default(autoincrement())
  name          String  @unique
  category      String  // platform, subject, pedagogical, accessibility
  color         String? // Hex color for UI
  links         LinkTag[]
  
  createdAt     DateTime @default(now())
}

model LinkTag {
  linkId        Int
  tagId         Int
  link          Link   @relation(fields: [linkId], references: [id], onDelete: Cascade)
  tag           Tag    @relation(fields: [tagId], references: [id], onDelete: Cascade)
  
  @@id([linkId, tagId])
  @@index([tagId])
}
```

## Frontend Modifications

### File Structure for UI Components

```
apps/web/src/
├── components/
│   ├── ui/                          # Shadcn/ui components
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── ...
│   ├── Layout/
│   │   ├── Sidebar.tsx
│   │   ├── Header.tsx
│   │   └── Navigation.tsx
│   ├── Link/                        # Link-related components
│   │   ├── LinkCard.tsx
│   │   ├── LinkForm.tsx             # ★ EDIT FOR XR FIELDS
│   │   └── LinkViewer.tsx
│   └── Collection/
│       ├── CollectionList.tsx
│       └── CollectionForm.tsx
├── pages/
│   ├── api/v1/                      # API routes
│   │   ├── auth/
│   │   ├── links/
│   │   ├── collections/
│   │   └── xr/                      # ★ CREATE FOR XR API
│   ├── _app.tsx                     # App wrapper
│   ├── _document.tsx                # Document structure
│   ├── index.tsx                    # Home page
│   └── links/                       # Link pages
├── hooks/                           # Custom React hooks
├── utils/                           # Utility functions
├── lib/                             # Library code
└── styles/
    ├── globals.css                  # Global styles
    └── layout.module.css
```

### Task: Add XR Fields to Link Form Component

File: `apps/web/src/components/Link/LinkForm.tsx`

```typescript
// Add these field groups to the form JSX

<fieldset className="xr-metadata">
  <legend>XR Educational Information</legend>
  
  {/* XR Platform Selection */}
  <div className="form-group">
    <label htmlFor="xrPlatform">XR Platform</label>
    <select
      id="xrPlatform"
      name="xrPlatform"
      value={formData.xrPlatform || ''}
      onChange={(e) => setFormData({...formData, xrPlatform: e.target.value})}
    >
      <option value="">Select Platform</option>
      <option value="quest-3">Meta Quest 3</option>
      <option value="quest-pro">Meta Quest Pro</option>
      <option value="apple-vision-pro">Apple Vision Pro</option>
      <option value="webxr">WebXR (Browser)</option>
      <option value="pico">Pico</option>
      <option value="vive">HTC Vive</option>
      <option value="other">Other</option>
    </select>
  </div>

  {/* Pedagogical Goal */}
  <div className="form-group">
    <label htmlFor="pedagogicalGoal">Learning Objective</label>
    <textarea
      id="pedagogicalGoal"
      name="pedagogicalGoal"
      placeholder="What learning outcomes does this resource support?"
      value={formData.pedagogicalGoal || ''}
      onChange={(e) => setFormData({...formData, pedagogicalGoal: e.target.value})}
    />
  </div>

  {/* Subject Area */}
  <div className="form-group">
    <label htmlFor="subjectArea">Subject Area</label>
    <select
      id="subjectArea"
      name="subjectArea"
      value={formData.subjectArea || ''}
      onChange={(e) => setFormData({...formData, subjectArea: e.target.value})}
    >
      <option value="">Select Subject</option>
      <option value="science">Science & Physics</option>
      <option value="mathematics">Mathematics</option>
      <option value="languages">Languages</option>
      <option value="history">History & Geography</option>
      <option value="arts">Arts & Design</option>
      <option value="vocational">Vocational Training</option>
    </select>
  </div>

  {/* Age/Grade Targeting */}
  <div className="form-group">
    <label htmlFor="targetAgeMin">Target Age/Grade Range</label>
    <div className="age-range">
      <input
        type="number"
        placeholder="Min age"
        value={formData.targetAgeMin || ''}
        onChange={(e) => setFormData({...formData, targetAgeMin: parseInt(e.target.value)})}
      />
      <span>to</span>
      <input
        type="number"
        placeholder="Max age"
        value={formData.targetAgeMax || ''}
        onChange={(e) => setFormData({...formData, targetAgeMax: parseInt(e.target.value)})}
      />
    </div>
  </div>

  {/* Duration */}
  <div className="form-group">
    <label htmlFor="duration">Typical Duration (minutes)</label>
    <input
      type="number"
      id="duration"
      placeholder="Duration"
      value={formData.duration || ''}
      onChange={(e) => setFormData({...formData, duration: parseInt(e.target.value)})}
    />
  </div>

  {/* Cost */}
  <div className="form-group">
    <label htmlFor="costType">Cost Type</label>
    <select
      id="costType"
      name="costType"
      value={formData.costType || ''}
      onChange={(e) => setFormData({...formData, costType: e.target.value})}
    >
      <option value="">Select</option>
      <option value="free">Free</option>
      <option value="freemium">Freemium</option>
      <option value="paid">Paid</option>
    </select>
  </div>

  {/* Accessibility */}
  <div className="form-group">
    <label htmlFor="accessibilityNotes">Accessibility Features</label>
    <textarea
      id="accessibilityNotes"
      placeholder="e.g., Hand tracking available, Wheelchair accessible"
      value={formData.accessibilityNotes || ''}
      onChange={(e) => setFormData({...formData, accessibilityNotes: e.target.value})}
    />
  </div>
</fieldset>
```

### Adding TypeScript Types

File: `apps/web/src/types/xr.ts` (create new file)

```typescript
export interface XrMetadata {
  xrPlatform?: string
  xrCategory?: string
  pedagogicalGoal?: string
  pedagogicalType?: string
  subjectArea?: string
  targetAgeMin?: number
  targetAgeMax?: number
  duration?: number
  costType?: string
  accessibilityNotes?: string
  requiredHardware?: string
}

export interface XrLink {
  id: number
  url: string
  title?: string
  xr: XrMetadata
}

export enum XrPlatform {
  QUEST_3 = 'quest-3',
  QUEST_PRO = 'quest-pro',
  APPLE_VISION_PRO = 'apple-vision-pro',
  WEBXR = 'webxr',
  PICO = 'pico',
  VR_OTHER = 'other'
}

export enum SubjectArea {
  SCIENCE = 'science',
  MATHEMATICS = 'mathematics',
  LANGUAGES = 'languages',
  HISTORY = 'history',
  ARTS = 'arts',
  VOCATIONAL = 'vocational'
}
```

## API Extensions

### Creating XR-Specific API Endpoints

File: `apps/web/src/pages/api/v1/xr/statistics.ts` (create new file)

```typescript
import { NextApiRequest, NextApiResponse } from 'next'
import { getServerSession } from 'next-auth/next'
import { prisma } from '@linkwarden/prisma'

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // Only GET requests allowed
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' })
  }

  try {
    // Get current user session
    const session = await getServerSession(req, res)
    if (!session?.user?.id) {
      return res.status(401).json({ error: 'Unauthorized' })
    }

    // Get XR content statistics
    const stats = await prisma.link.groupBy({
      by: ['xrPlatform'],
      where: {
        userId: parseInt(session.user.id as string),
      },
      _count: true,
    })

    // Get subject area distribution
    const subjects = await prisma.link.groupBy({
      by: ['subjectArea'],
      where: {
        userId: parseInt(session.user.id as string),
      },
      _count: true,
    })

    // Get age group distribution
    const ageGroups = await prisma.link.findMany({
      where: {
        userId: parseInt(session.user.id as string),
      },
      select: {
        targetAgeMin: true,
        targetAgeMax: true,
      },
    })

    return res.status(200).json({
      platforms: stats,
      subjects: subjects,
      ageDistribution: ageGroups,
    })
  } catch (error) {
    console.error('XR statistics error:', error)
    return res.status(500).json({ error: 'Internal server error' })
  }
}
```

File: `apps/web/src/pages/api/v1/xr/search.ts` (create new file)

```typescript
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' })
  }

  const { platform, subject, ageMin, ageMax, costType } = req.body

  try {
    const filters: any = {}
    
    if (platform) filters.xrPlatform = platform
    if (subject) filters.subjectArea = subject
    if (costType) filters.costType = costType
    
    if (ageMin || ageMax) {
      filters.AND = [
        { targetAgeMin: { lte: ageMax || 100 } },
        { targetAgeMax: { gte: ageMin || 0 } }
      ]
    }

    const results = await prisma.link.findMany({
      where: filters,
      include: { collection: true },
      take: 50,
    })

    return res.status(200).json(results)
  } catch (error) {
    return res.status(500).json({ error: 'Search failed' })
  }
}
```

## Styling & Branding

### Tailwind CSS Configuration

File: `apps/web/tailwind.config.ts`

```typescript
module.exports = {
  theme: {
    extend: {
      colors: {
        // XR Education brand colors
        xr: {
          50: '#f0f9ff',    // Lightest
          100: '#e0f2fe',
          500: '#8B5CF6',   // XR Purple
          900: '#5B21B6',   // Darkest
        },
        education: {
          50: '#eff6ff',
          500: '#2563EB',   // Education Blue
          900: '#1e3a8a',
        },
        pedagogical: {
          interactive: '#10B981',  // Green
          simulation: '#F59E0B',   // Amber
          gameBased: '#EF4444',    // Red
          exploration: '#06B6D4',  // Cyan
        }
      },
      fontFamily: {
        'display': ['Inter', 'sans-serif'], // Headings
        'body': ['Inter', 'sans-serif'],    // Body text
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
  ],
}
```

### Global Styles

File: `apps/web/src/styles/xr-custom.css`

```css
/* XR Educational Theme */

:root {
  --color-xr-primary: #8B5CF6;
  --color-xr-secondary: #2563EB;
  --color-xr-accent: #10B981;
  --color-xr-warning: #F59E0B;
}

/* XR Metadata Display */
.xr-badge {
  @apply inline-flex items-center gap-2 px-3 py-1 rounded-full text-sm font-medium;
}

.xr-badge.platform {
  @apply bg-blue-100 text-blue-900;
}

.xr-badge.subject {
  @apply bg-purple-100 text-purple-900;
}

.xr-badge.pedagogical {
  @apply bg-green-100 text-green-900;
}

.xr-metadata-section {
  @apply bg-gray-50 dark:bg-gray-900 rounded-lg p-4 space-y-3;
}

.xr-metadata-item {
  @apply flex items-start gap-3;
}

.xr-metadata-label {
  @apply font-semibold text-gray-700 dark:text-gray-300 w-24;
}

.xr-metadata-value {
  @apply text-gray-900 dark:text-gray-100 flex-1;
}

/* Age Range Display */
.age-badge {
  @apply inline-flex items-center gap-1 px-2 py-1 rounded text-sm font-medium;
  background: linear-gradient(135deg, var(--color-xr-primary), var(--color-xr-secondary));
  color: white;
}

/* Platform Icons */
.platform-icon {
  @apply w-5 h-5 inline-block;
}

.platform-icon.quest {
  color: #1f2937;
}

.platform-icon.apple {
  color: #555555;
}

.platform-icon.webxr {
  color: #FF6B35;
}
```

## Testing & Deployment

### Unit Testing

File: `apps/web/src/components/Link/__tests__/LinkForm.test.tsx`

```typescript
import { render, screen, userEvent } from '@testing-library/react'
import { LinkForm } from '../LinkForm'

describe('LinkForm with XR Fields', () => {
  test('renders XR metadata section', () => {
    render(<LinkForm />)
    expect(screen.getByText('XR Educational Information')).toBeInTheDocument()
  })

  test('accepts XR platform selection', async () => {
    render(<LinkForm />)
    const select = screen.getByDisplayValue('Select Platform')
    await userEvent.selectOption(select, 'quest-3')
    expect(select).toHaveValue('quest-3')
  })

  test('accepts educational metadata', async () => {
    render(<LinkForm />)
    const goalInput = screen.getByPlaceholderText(/learning outcomes/i)
    await userEvent.type(goalInput, 'Learn about photosynthesis')
    expect(goalInput).toHaveValue('Learn about photosynthesis')
  })
})
```

### Running Tests

```bash
# Run all tests
npm run test

# Run tests in watch mode
npm run test:watch

# Generate coverage report
npm run test:coverage
```

### Building for Production

```bash
# Build web application
npm run web:build

# Start production server
npm run web:start

# Or with Docker
docker-compose build
docker-compose up -d
```

## Common Customization Scenarios

### Scenario 1: Add a New XR Platform

**Step 1:** Add to database enumeration (optional)

```prisma
enum XrPlatform {
  QUEST_3
  QUEST_PRO
  APPLE_VISION_PRO
  WEBXR
  PICO
  NEW_PLATFORM
}

model Link {
  xrPlatform    XrPlatform?
```

**Step 2:** Update frontend select options

```typescript
const XR_PLATFORMS = [
  { value: 'quest-3', label: 'Meta Quest 3' },
  { value: 'apple-vision-pro', label: 'Apple Vision Pro' },
  { value: 'new-platform', label: 'New Platform' },  // Add here
]
```

**Step 3:** Create migration

```bash
npx prisma migrate dev --name add_new_platform
```

### Scenario 2: Add Teacher/Student Role Distinction

```prisma
enum UserRole {
  ADMIN    // Can manage everything
  CURATOR  // Can verify and curate content
  EDUCATOR // Can create and share collections
  STUDENT  // Read-only by default
}

model User {
  id            Int     @id @default(autoincrement())
  email         String  @unique
  name          String?
  role          UserRole @default(EDUCATOR)
  // ... other fields
}
```

### Scenario 3: Add Custom Metadata Validation

File: `packages/utils/src/validators/xr.ts` (create new)

```typescript
export function validateXrMetadata(data: any) {
  const errors: string[] = []

  if (data.targetAgeMin && data.targetAgeMax) {
    if (data.targetAgeMin > data.targetAgeMax) {
      errors.push('Minimum age cannot be greater than maximum age')
    }
  }

  if (data.duration && data.duration <= 0) {
    errors.push('Duration must be positive')
  }

  if (data.costAmount && data.costType !== 'paid') {
    errors.push('Cost amount only applies to paid resources')
  }

  return { valid: errors.length === 0, errors }
}
```

## Deployment Checklist

Before deploying customizations to production:

- [ ] All tests pass locally
- [ ] Database migrations created and tested
- [ ] Schema changes reviewed
- [ ] UI components tested in all breakpoints
- [ ] API endpoints tested with valid/invalid data
- [ ] Documentation updated
- [ ] Environment variables configured
- [ ] Backup created before migration
- [ ] Deployment plan documented
- [ ] Rollback procedure prepared

---

## Resources

- [Prisma Documentation](https://www.prisma.io/docs/)
- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [Tailwind CSS](https://tailwindcss.com)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---

**Last Updated:** March 10, 2026
