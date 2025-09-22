# QuiverRack Mono-repo Structure Specification

## Repository Architecture

QuiverRack will use a mono-repo architecture to manage multiple applications and shared code. This approach enables code sharing, consistent tooling, and simplified dependency management across our ecosystem.

### Mono-repo Structure

```
quiverrack/
├── apps/
│   ├── web/                    # Main client web application (Next.js)
│   │   ├── app/               # Next.js 13+ app directory
│   │   ├── components/        # Web-specific components
│   │   ├── public/           
│   │   └── package.json
│   │
│   └── admin/                  # Admin dashboard application (Next.js)
│       ├── app/               # Next.js 13+ app directory
│       ├── components/        # Admin-specific components
│       ├── public/
│       └── package.json
│
├── packages/
│   ├── ui/                     # Shared UI component library
│   │   ├── components/        # Reusable components
│   │   ├── styles/           # Shared styles/design tokens
│   │   └── package.json
│   │
│   ├── database/              # Database schema and migrations
│   │   ├── migrations/       # Supabase migrations
│   │   ├── types/           # Generated TypeScript types
│   │   ├── seed/            # Seed data
│   │   └── package.json
│   │
│   ├── supabase/             # Supabase client and functions
│   │   ├── client/          # Configured Supabase client
│   │   ├── functions/       # Edge functions
│   │   └── package.json
│   │
│   ├── shared/               # Shared utilities and business logic
│   │   ├── constants/       # App-wide constants
│   │   ├── types/          # Shared TypeScript types
│   │   ├── utils/          # Utility functions
│   │   ├── hooks/          # Shared React hooks
│   │   └── package.json
│   │
│   └── config/               # Shared configuration
│       ├── eslint/         # ESLint config
│       ├── typescript/     # TypeScript config
│       └── package.json
│
├── docs/                      # Project documentation
├── scripts/                   # Build and deployment scripts
├── .github/                   # GitHub Actions workflows
├── turbo.json                # Turborepo configuration
├── package.json              # Root package.json
├── pnpm-workspace.yaml       # PNPM workspace config
└── README.md
```

### Technology Choices

#### Package Manager: PNPM
- Efficient disk space usage through hard links
- Strict dependency resolution
- Built-in workspace support
- Fast installation times

#### Build Tool: Turborepo
- Intelligent build caching
- Parallel execution
- Remote caching capabilities
- Dependency graph awareness

### Package Descriptions

#### `apps/web`
The main client-facing application where users create and manage their ski profiles.
- Next.js 14 with App Router
- Tailwind CSS for styling
- Supabase Auth integration
- Vercel deployment

#### `apps/admin`
Administrative dashboard for managing the ski database and user moderation.
- Next.js 14 with App Router
- Role-based access control
- Content moderation tools
- Analytics dashboard
- Ski database management

#### `packages/ui`
Shared component library following the QuiverRack design system.
- React components
- Tailwind CSS utilities
- Storybook for component documentation
- Design tokens for consistent theming

#### `packages/database`
Centralized database management and type generation.
- Supabase migrations
- TypeScript types generated from database schema
- Seed data for development
- Database utilities

#### `packages/supabase`
Shared Supabase client configuration and edge functions.
- Configured Supabase client with proper types
- Shared edge functions
- RLS helpers
- Auth utilities

#### `packages/shared`
Business logic and utilities shared across applications.
- User profile validation
- Ski data processing
- Image handling utilities
- Common React hooks
- Type definitions

### Development Workflow

```bash
# Install dependencies for all packages
pnpm install

# Development commands
pnpm dev              # Start all apps in development mode
pnpm dev:web          # Start only web app
pnpm dev:admin        # Start only admin app

# Build commands
pnpm build            # Build all apps and packages
pnpm build:web        # Build only web app
pnpm build:admin      # Build only admin app

# Testing
pnpm test             # Run all tests
pnpm test:unit        # Run unit tests
pnpm test:e2e         # Run e2e tests

# Type checking
pnpm typecheck        # Check types across all packages

# Linting
pnpm lint             # Lint all packages
pnpm lint:fix         # Fix linting issues
```

### Shared Code Examples

#### Using Shared UI Components
```typescript
// In apps/web/app/profile/page.tsx
import { Button, SkiCard, ProfileAvatar } from '@quiverrack/ui';
import { formatSkiSpecs } from '@quiverrack/shared/utils';
```

#### Using Shared Types
```typescript
// In apps/admin/app/skis/page.tsx
import type { Ski, UserProfile } from '@quiverrack/shared/types';
import { useSupabase } from '@quiverrack/supabase/client';
```

### Environment Variables

Each app maintains its own `.env.local` file, but shared configuration can be referenced:

```
# apps/web/.env.local
NEXT_PUBLIC_SUPABASE_URL=your-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-key
NEXT_PUBLIC_APP_URL=https://quiverrack.com

# apps/admin/.env.local
NEXT_PUBLIC_SUPABASE_URL=your-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-key
NEXT_PUBLIC_APP_URL=https://admin.quiverrack.com
SUPABASE_SERVICE_ROLE_KEY=your-service-key
```

### CI/CD Configuration

GitHub Actions workflow for mono-repo:

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm typecheck
      - run: pnpm lint
      - run: pnpm test
      - run: pnpm build
```

### Deployment Strategy

- **Web App**: Deployed to Vercel from `apps/web`
- **Admin App**: Deployed to Vercel from `apps/admin` on separate domain
- **Edge Functions**: Deployed with Supabase CLI from `packages/supabase/functions`

### Future Considerations

#### Mobile App Integration
When adding a mobile app, consider:

**Option 1: Include in Mono-repo**
```
apps/
├── mobile/           # React Native app
│   ├── src/
│   ├── ios/
│   ├── android/
│   └── package.json
```

Benefits:
- Shared types and utilities
- Consistent API interfaces
- Unified CI/CD

**Option 2: Separate Repository**
Create `quiverrack-mobile` repository that consumes published packages from the mono-repo.

Benefits:
- Cleaner separation of concerns
- Different release cycles
- Platform-specific tooling

### Migration Path

1. **Phase 1**: Set up mono-repo structure with web app
2. **Phase 2**: Extract shared components to packages
3. **Phase 3**: Add admin app
4. **Phase 4**: Publish internal packages (if needed for mobile)
5. **Phase 5**: Integrate or link mobile app

### Best Practices

1. **Keep packages focused**: Each package should have a single, clear purpose
2. **Minimize circular dependencies**: Use dependency injection where needed
3. **Version together**: Use synchronized versioning for simplicity
4. **Document interfaces**: Clearly document public APIs for packages
5. **Test in isolation**: Each package should have its own test suite
6. **Use TypeScript**: Leverage TypeScript for type safety across packages
