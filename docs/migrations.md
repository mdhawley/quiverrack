# Database Migrations Documentation

This document explains the database migrations added during development and their purposes.

## Migration 008: Add Avatar and Location to Users

**File:** `packages/database/src/migrations/008_add_avatar_location_to_users.sql`

### Purpose
Add support for user avatars and location information in profiles.

### Changes
```sql
ALTER TABLE users 
ADD COLUMN IF NOT EXISTS avatar_url TEXT,
ADD COLUMN IF NOT EXISTS location VARCHAR(100);
```

### Notes
- These columns turned out to already exist in the production database
- The `IF NOT EXISTS` clause prevented errors
- `avatar_url` stores the full public URL from Supabase Storage
- `location` is a free-text field limited to 100 characters

## Migration 009: Fix Storage Policies

**File:** `packages/database/src/migrations/009_fix_storage_policies.sql`

### Purpose
Update storage bucket RLS policies to support anonymous user uploads.

### Changes
1. Dropped existing avatar storage policies
2. Recreated policies to accept both 'authenticated' and 'anon' roles
3. Applied same changes to quiver-photos bucket for consistency

### Key Policy Changes
```sql
-- Old policy (failed for anonymous users)
AND auth.role() = 'authenticated'

-- New policy (works for both)
AND (auth.role() = 'authenticated' OR auth.role() = 'anon')
```

### Implementation Note
While this migration was created, the actual solution implemented was to use a server-side upload endpoint that bypasses RLS entirely using the admin client. This provides better security and control.

## Migration Patterns

### Table Naming Confusion
During development, we discovered a mismatch between expected and actual table names:
- **Expected:** `user_profiles` (from migration 001)
- **Actual:** `users` (what the application uses)

This suggests either:
1. The initial migrations were not run in production
2. Different table names were used in production setup
3. There's a mapping layer we're not aware of

### Terrain Preferences Structure
The `terrain_preferences` table (migration 002) references `user_profiles(id)`, but since the app uses `users` table, there's actually a `user_terrain_preferences` table that references `users(id)`.

## Best Practices for Future Migrations

1. **Always use IF NOT EXISTS**: Prevents errors if columns/tables already exist
2. **Check actual database state**: Don't assume migrations match production
3. **Consider backwards compatibility**: Existing data should not break
4. **Document assumptions**: Note which tables/columns are expected
5. **Test rollback strategy**: Ensure migrations can be safely reverted

## Running Migrations

To run migrations locally:
```bash
cd packages/database
pnpm migrate
```

For production, migrations should be run through the Supabase dashboard or CLI:
```bash
supabase db push
```

## Rollback Procedures

### Rolling back 008
```sql
ALTER TABLE users 
DROP COLUMN IF EXISTS avatar_url,
DROP COLUMN IF EXISTS location;
```

### Rolling back 009
Restore the original storage policies by running the original policy creation from migration 006.

## Future Migration Needs

1. **Consolidate table references**: Decide on `users` vs `user_profiles`
2. **Add indexes**: Consider indexes on frequently queried fields
3. **Data cleanup**: Remove unused tables/columns
4. **Audit trail**: Add tables for tracking profile changes
5. **Performance optimization**: Analyze query patterns and optimize