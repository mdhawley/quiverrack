# Staging Environment Automation

This document describes the automated setup process for creating and managing a Supabase staging environment for QuiverRack.

## Overview

The staging environment automation provides a one-command setup for creating a complete staging instance of QuiverRack, including:
- Supabase project creation
- Database migrations
- Seed data
- Storage buckets
- OAuth configuration
- Environment files
- CI/CD pipeline

## Prerequisites

### Required Software
1. **Supabase CLI**: Install globally
   ```bash
   npm install -g supabase
   ```

2. **Node.js & pnpm**: Required for running scripts
   ```bash
   npm install -g pnpm
   ```

3. **jq**: JSON processor for bash scripts
   ```bash
   # macOS
   brew install jq
   
   # Ubuntu/Debian
   sudo apt-get install jq
   ```

### Authentication
1. Login to Supabase CLI:
   ```bash
   supabase login
   ```
   This will open your browser for authentication.

2. Get your organization ID:
   ```bash
   supabase orgs list
   ```

### Environment Variables
Set these before running the setup (all are optional):

```bash
# Optional - Script will prompt if not set
export SUPABASE_ORG_ID="your-org-id"              # Your organization ID (or select interactively)
export STAGING_DB_PASSWORD="secure-password"       # Or let script generate one

# Optional - For OAuth configuration
export GOOGLE_CLIENT_ID="your-google-client-id"
export GOOGLE_CLIENT_SECRET="your-google-secret"
```

**Note**: The Supabase access token is handled automatically by the CLI after running `supabase login`. You don't need to set it manually.

## Quick Start

### 1. Run the Automated Setup

```bash
npm run staging:setup
```

This single command will:
- Create a new Supabase project named "quiverrack-staging"
- Link the project to your local development
- Apply all database migrations
- Seed the database with test data
- Configure storage buckets
- Generate environment files
- Provide OAuth configuration instructions

### 2. Manual Steps After Setup

#### Configure OAuth Providers
1. Go to your [Supabase Dashboard](https://supabase.com/dashboard)
2. Navigate to your staging project
3. Go to Authentication > Providers
4. Configure Google OAuth with the credentials from your environment
5. Add redirect URLs from `scripts/staging-config.json`

#### Set Up Vercel Staging
1. Create a new Vercel project or add staging environment
2. Configure environment variables:
   - Copy values from `apps/web/.env.staging`
   - Copy values from `apps/admin/.env.staging`
3. Connect to `staging` branch for automatic deployments

#### Configure GitHub Secrets
Add these secrets to your GitHub repository:
```
SUPABASE_ACCESS_TOKEN
STAGING_PROJECT_ID
STAGING_DB_PASSWORD
STAGING_SUPABASE_URL
STAGING_SUPABASE_ANON_KEY
STAGING_SUPABASE_SERVICE_KEY
VERCEL_TOKEN
VERCEL_ORG_ID
VERCEL_WEB_PROJECT_ID
VERCEL_ADMIN_PROJECT_ID
```

## File Structure

### Configuration Files
- `scripts/staging-config.json` - Main configuration for staging environment
- `scripts/setup-staging-environment.sh` - Main orchestration script
- `scripts/generate-env-files.ts` - Generates .env files
- `scripts/setup-staging-storage.ts` - Configures storage buckets
- `scripts/configure-staging-oauth.ts` - OAuth setup helper

### Generated Files
- `apps/web/.env.staging` - Web app staging environment
- `apps/admin/.env.staging` - Admin app staging environment
- `staging-project-info.json` - Project details for reference
- `.github/staging-secrets.md` - GitHub secrets reference (don't commit!)

### Database Files
- `supabase/seeds/staging-seed.sql` - Staging-specific seed data
- `supabase/migrations/` - All database migrations

## Available Commands

After setup, use these commands to manage your staging environment:

```bash
# Apply new migrations
npm run staging:migrate

# Seed database
npm run staging:seed

# Reset database (drops and recreates)
npm run staging:reset

# Check project status
npm run staging:status

# Check migration differences
npm run staging:diff
```

**Note**: All commands use `--project-ref` to target the staging environment without changing your local link. Your production link remains unchanged.

### Using a Different Staging Project

If you need to use a different staging project ID:

```bash
# Set the project ID for this session
export STAGING_PROJECT_ID=your-project-id

# Or use it inline
STAGING_PROJECT_ID=your-project-id npm run staging:migrate
```

## CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/staging-deploy.yml`) automatically:
1. Runs tests on push to `staging` branch
2. Applies database migrations
3. Deploys web app to Vercel
4. Deploys admin app to Vercel
5. Seeds database (if needed)
6. Sends deployment notifications

### Manual Deployment

Trigger deployment manually:
```bash
git push origin main:staging
```

Or use GitHub Actions UI to trigger workflow with options.

## Configuration Details

### Project Settings
Defined in `scripts/staging-config.json`:
- **Region**: us-east-1 (configurable)
- **Plan**: free (upgradeable)
- **Auth**: Anonymous + Email + Google OAuth
- **Storage**: 2 public buckets (avatars, quiver-photos)

### Feature Flags
Default staging features:
- `FEATURE_AI_ASSISTANT`: invited-groups
- `FEATURE_RESORT_CONDITIONS`: off
- `FEATURE_WEATHER_WIDGET`: public

### Test Data
Staging seed includes:
- App settings and feature flags
- Rate limiting configuration
- System settings marking it as staging
- Notification templates for testing

## Troubleshooting

### Setup Script Fails

1. **"Supabase CLI not found"**
   - Install: `npm install -g supabase`

2. **"Not logged in to Supabase"**
   - Run: `supabase login`

3. **"Project already exists"**
   - Choose to use existing or delete old project first

4. **"Failed to apply migrations"**
   - Check migration files are valid SQL
   - Ensure database password is correct

### Environment Issues

1. **Can't connect to staging from admin tools**
   - Check `STAGING_*` variables in `apps/admin/.env.local`
   - Restart admin dev server

2. **OAuth not working**
   - Verify redirect URLs in Supabase Dashboard
   - Check Google Cloud Console settings
   - Ensure client ID/secret are correct

### Database Issues

1. **Seed data fails**
   - May be okay if data already exists
   - Check for unique constraint violations
   - Run `npm run staging:reset` to start fresh

2. **Migrations out of sync**
   - Run `supabase db push --linked --dry-run` to preview
   - Use `supabase db reset --linked` for clean slate

## Best Practices

### Security
- Never commit `.env` files with real credentials
- Use strong passwords for staging database
- Rotate service keys periodically
- Limit staging access to team members

### Data Management
- Use anonymized/fake data in staging
- Don't copy production data directly
- Regular cleanup of old test data
- Monitor storage usage

### Testing
- Test all migrations in staging before production
- Verify OAuth flows completely
- Test rate limiting and error handling
- Validate mobile responsiveness

### Maintenance
- Keep staging in sync with production schema
- Update seed data as features evolve
- Document any manual configuration
- Regular staging environment refreshes

## Environment Promotion

### Staging to Production Workflow

1. **Test in Staging**
   - All features thoroughly tested
   - Performance validated
   - Security reviewed

2. **Prepare Production**
   - Export migrations from staging
   - Review configuration differences
   - Prepare production secrets

3. **Deploy to Production**
   - Apply migrations
   - Update environment variables
   - Deploy applications
   - Monitor for issues

## Cleanup

To completely remove staging environment:

1. **Delete Supabase Project**
   ```bash
   supabase projects delete [staging-project-id]
   ```

2. **Remove Local Files**
   ```bash
   rm staging-project-info.json
   rm apps/web/.env.staging
   rm apps/admin/.env.staging
   ```

3. **Remove from Vercel**
   - Delete staging environment in Vercel dashboard

4. **Clean GitHub Secrets**
   - Remove staging-specific secrets

## Support

For issues or questions:
1. Check Supabase logs: `supabase logs --project-ref [staging-id]`
2. Review GitHub Actions logs for deployment issues
3. Consult Supabase documentation
4. Check QuiverRack project documentation

## Appendix: Manual Setup

If automated setup fails, follow these manual steps:

1. Create project in Supabase Dashboard
2. Get project credentials
3. Link locally: `supabase link --project-ref [id]`
4. Apply migrations: `supabase db push`
5. Configure storage buckets manually
6. Set up OAuth providers
7. Create .env files manually
8. Configure Vercel projects
9. Set up GitHub secrets
10. Test deployment pipeline

---

Last Updated: 2024
Version: 1.0.0