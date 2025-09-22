# Development Notes

This document contains implementation details and lessons learned during development.

## Profile Editing Feature Implementation

### Overview
Implemented a comprehensive profile editing system that allows users to update their profile information, upload avatars, and manage terrain preferences.

### Components Created
1. **ProfileEditModal** (`/components/dashboard/profile-edit-modal.tsx`)
   - React Hook Form with Zod validation
   - Multi-field editing interface
   - Avatar preview functionality
   - Terrain preference multi-select

2. **ProfileHeader Updates**
   - Added avatar display with fallback initials
   - Integrated edit modal trigger
   - Shows location field when available

### Key Implementation Details
- Form state managed with React Hook Form
- Validation using Zod schemas
- Optimistic UI updates after save
- Server-side avatar upload for reliability

## Avatar Upload System Architecture

### Problem
Anonymous users created via the profile creation API couldn't upload avatars due to Supabase RLS policies expecting authenticated users.

### Initial Approach (Failed)
```typescript
// Client-side upload using Supabase client
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(filePath, file);
// Failed with 400 error for anonymous users
```

### Solution: Server-Side Upload
Created dedicated upload endpoint that uses admin privileges:

```typescript
// POST /api/v1/upload/avatar
const { data, error } = await supabaseAdmin.storage
  .from('avatars')
  .upload(filePath, buffer, {
    contentType: file.type,
    cacheControl: '3600',
    upsert: true,
  });
```

### Implementation Notes
- File validation (type, size) on server
- FormData API for file transfer
- Returns public URL for immediate use
- Error handling for upload failures

## Terrain Preferences Update Logic

### Database Structure
- Table: `user_terrain_preferences`
- Columns: `user_id` (UUID), `terrain_type_id` (integer)
- Many-to-many relationship with compound unique key

### Terrain Type Mapping
```typescript
const TERRAIN_TYPE_ID_MAP = {
  'Groomers': 1,
  'Moguls': 2,
  'Powder': 3,
  'Park': 4,
  'Trees': 5,
  'Steeps': 6,
  'Backcountry': 7
};
```

### Update Process
1. Delete all existing preferences for user
2. Insert new preferences from array
3. Use transaction-like approach (though not true DB transaction)

### API Integration
- GET endpoints JOIN terrain preferences table
- Convert IDs to strings for API responses
- PATCH endpoint handles array of terrain names

## Troubleshooting Guide

### Issue: Profile Update Returns 500 Error
**Symptom**: PATCH request fails with "terrain_preferences column not found"

**Cause**: Trying to update terrain_preferences as a column when it's actually a separate table

**Solution**: Remove terrain_preferences from direct update, handle separately

### Issue: Avatar Upload Fails with 400
**Symptom**: Client-side upload returns 400 error

**Cause**: RLS policies require authenticated user, but we have anonymous users

**Solution**: Use server-side upload endpoint with admin client

### Issue: Public Profile Shows Stale Data
**Symptom**: Updates don't appear on public profile immediately

**Cause**: Unknown - possibly caching or different query path

**Status**: Unresolved - dashboard API works correctly

### Issue: Display Names Auto-Generated
**Symptom**: All profiles get "SkierXXXXXX" display names

**Cause**: Hardcoded in profile creation API

**Consideration**: Could make display_name optional in creation

## Best Practices Discovered

### 1. Server-Side Operations for Anonymous Users
When dealing with anonymous users and Supabase, prefer server-side operations using the admin client to bypass RLS.

### 2. Separate Complex Relationships
Don't try to update many-to-many relationships as part of the main entity update. Handle them separately.

### 3. Consistent Field Naming
Be aware of field name differences between database and API (e.g., skier_level vs skier_ability).

### 4. Explicit Error Handling
Always provide detailed error messages in API responses to aid debugging.

### 5. Form Validation
Use Zod for form validation to ensure type safety and good error messages.

## Future Improvements

1. **True Database Transactions**: Implement proper transactions for terrain preference updates
2. **Cache Invalidation**: Fix public profile caching issue
3. **Optional Display Names**: Allow users to create profiles without display names
4. **Bulk Operations**: Add bulk update capabilities for terrain preferences
5. **Image Optimization**: Add server-side image resizing for avatars
6. **Audit Trail**: Track profile changes for moderation purposes