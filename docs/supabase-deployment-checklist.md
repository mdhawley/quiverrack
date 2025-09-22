# Supabase Production Deployment Checklist

Quick reference checklist for deploying QuiverRack to Supabase production.

## 🚀 Pre-Deployment (30 mins before)

### Backup & Verification
- [ ] Run backup: `./scripts/backup/backup-all.sh`
- [ ] Verify backup completed: `ls -la backups/latest/`
- [ ] Run pre-deploy check: `./scripts/dev-workflow/pre-deploy-check.sh`
- [ ] Ensure on production branch: `git checkout production`
- [ ] Pull latest changes: `git pull origin production`

### Environment Check
- [ ] Supabase CLI installed: `supabase --version`
- [ ] Logged in to Supabase: `supabase login`
- [ ] Production project created in dashboard
- [ ] Have all credentials ready:
  - [ ] Project Reference ID
  - [ ] Database password
  - [ ] Service role key
  - [ ] Anon key

## 📦 Deployment Steps

### 1. Database Migration (10-15 mins)
```bash
# Link to production
supabase link --project-ref <PROJECT_REF>

# Option A: Squashed migration (first deploy)
./scripts/backup/squash-migrations.sh
rm -rf supabase/migrations/*
cp migration-consolidation/001_initial_schema.sql supabase/migrations/

# Option B: Individual migrations (updates)
# Keep existing migrations

# Dry run
supabase db push --dry-run

# Apply migrations
supabase db push

# Verify
supabase db migrations list
```

### 2. Seed Data (2 mins)
```sql
-- Run in SQL Editor
-- Terrain types
INSERT INTO terrain_types (id, name) VALUES
(1,'Groomers'),(2,'Moguls'),(3,'Powder'),
(4,'Park'),(5,'Trees'),(6,'Steeps'),(7,'Backcountry')
ON CONFLICT DO NOTHING;

-- App settings
INSERT INTO app_settings (key, value, value_type) VALUES
('maintenance_mode','false','boolean'),
('allow_anonymous_signup','true','boolean'),
('ai_assistant_enabled','true','boolean')
ON CONFLICT DO NOTHING;
```

### 3. Edge Functions (2 mins)
```bash
# Deploy all functions
supabase functions deploy

# Verify
supabase functions list
```

### 4. Storage Buckets (1 min)
```sql
-- Verify buckets exist
SELECT id, name, public FROM storage.buckets;
-- Should show: avatars, quiver-images, avatar-images, chat-media, feed-media
```

### 5. Authentication (5 mins)
In Dashboard → Authentication:
- [ ] Enable Email Auth
- [ ] Enable Anonymous Sign-ins
- [ ] Disable email confirmation (MVP)
- [ ] Set JWT expiry: 3600
- [ ] Add Google OAuth credentials
- [ ] Set Site URL: `https://quiverrack.com`
- [ ] Add redirect URLs

### 6. Environment Variables (5 mins)
In Vercel/Platform:
```bash
NEXT_PUBLIC_SUPABASE_URL=https://<ref>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<anon-key>
SUPABASE_SERVICE_ROLE_KEY=<service-key>
NEXT_PUBLIC_APP_URL=https://quiverrack.com
```

In GitHub Secrets:
```bash
SUPABASE_ACCESS_TOKEN=<token>
PRODUCTION_PROJECT_ID=<ref>
PRODUCTION_DB_PASSWORD=<password>
```

## ✅ Post-Deployment Verification (10 mins)

### Database Checks
```sql
-- Check migrations
SELECT COUNT(*) FROM supabase_migrations.schema_migrations;

-- Check tables (should be 57+)
SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='public';

-- Check RLS policies
SELECT COUNT(*) FROM pg_policies WHERE schemaname='public';
```

### Functionality Tests
- [ ] Create anonymous user
- [ ] Sign up with email
- [ ] Create profile
- [ ] Upload avatar image
- [ ] Add ski to profile
- [ ] Add resort preference
- [ ] Create feed post
- [ ] Send message

### Monitoring
- [ ] Check Supabase logs for errors
- [ ] Monitor Edge Function logs
- [ ] Check storage bucket access
- [ ] Verify OAuth flow works

## 🚨 Quick Fixes

### Migration Failed
```bash
# Check error
supabase db migrations list

# Rollback if needed
# Use rollback SQL from migration file

# Retry with squashed migration
./scripts/backup/squash-migrations.sh
```

### Storage Not Working
```sql
-- Check bucket exists
SELECT * FROM storage.buckets WHERE name='avatars';

-- Check policies
SELECT * FROM pg_policies WHERE tablename='objects';
```

### Auth Issues
- Check Dashboard → Authentication → Settings
- Verify redirect URLs in URL Configuration
- Check OAuth provider configuration

### Functions Not Working
```bash
# Check deployment
supabase functions list

# Check logs
supabase functions logs <function-name>

# Redeploy specific function
supabase functions deploy <function-name>
```

## 🔄 Rollback Commands

### Full Rollback
```bash
# Restore from backup
./scripts/backup/restore-database.sh <timestamp>

# Revert code
git revert HEAD && git push
```

### Partial Rollback
```sql
-- Rollback specific migration
-- Use rollback SQL from migration file
-- Example:
DROP TABLE IF EXISTS new_table;
ALTER TABLE modified_table DROP COLUMN new_column;
```

## 📞 Escalation

If issues persist:
1. Check Supabase Dashboard logs
2. Review `/SUPABASE_DEPLOYMENT.md`
3. Contact team lead
4. Open Supabase support ticket

## 🎯 Success Criteria

Deployment is successful when:
- [ ] All migrations applied
- [ ] No errors in logs
- [ ] Core features working
- [ ] Monitoring active
- [ ] Backup created
- [ ] Team notified

---

**Time Estimate:** 30-45 minutes total
**Peak Hours:** Avoid 9am-12pm, 6pm-9pm EST
**Best Time:** 2am-5am EST (lowest traffic)