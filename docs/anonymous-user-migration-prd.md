# Anonymous User Migration Pattern

## Overview

This document describes the technical approach for preserving anonymous user data when users authenticate and claim their profiles. This pattern ensures seamless user experience by maintaining continuity of all user activity, preferences, and profile data during the authentication transition.

## Problem Statement

Users begin their journey anonymously, creating profiles and interacting with the platform before deciding to authenticate. When they authenticate, we need to preserve all their anonymous activity and data while transitioning them to an authenticated state that works with our Row Level Security (RLS) policies.

## Technical Approach

### Core Migration Strategy

Instead of updating the anonymous user ID to match the authenticated user ID, we **create a new profile** with the authenticated user's ID and **migrate all data** from the anonymous profile. This ensures:

- RLS policies function correctly (requiring `user_id = auth.uid()`)
- Authenticated user ID becomes the permanent identifier
- Complete preservation of anonymous user data and activity

### Migration Process

#### 1. Profile Claim Initiation
- User provides `claimToken` (links to anonymous profile) and `displayName`
- System validates authenticated user exists and email is confirmed
- Lookup anonymous profile using claim token, ensuring it's unclaimed

#### 2. Data Migration
**Primary Profile Creation:**
```sql
INSERT INTO users (
  id,                    -- authenticated user's ID
  display_name,          -- user-provided display name  
  email,                 -- from authenticated user
  first_name,            -- copied from anonymous profile
  last_name,             -- copied from anonymous profile
  age,                   -- copied from anonymous profile
  height,         -- copied from anonymous profile
  weight,            -- copied from anonymous profile
  location,              -- copied from anonymous profile
  skier_level,           -- copied from anonymous profile
  skiing_style,          -- copied from anonymous profile
  bio,                   -- copied from anonymous profile
  avatar_url,            -- copied from anonymous profile
  hero_image_url,        -- copied from anonymous profile
  is_claimed,            -- set to true
  is_public,             -- copied from anonymous profile
  claim_token            -- set to null
)
```

**Related Data Migration:**
- **Terrain Preferences**: Copy all `user_terrain_preferences` records
- **Resort Associations**: Copy all `user_resorts` records with frequency data
- **Equipment**: Copy all `user_skis` records with notes
- **Activity History**: Copy all `user_inputs` records (LLM interactions, costs, etc.)

#### 3. Cleanup
- Delete original anonymous profile
- Foreign key constraints with CASCADE handle automatic cleanup of orphaned related data

### Implementation Details

#### Database Function
```sql
CREATE OR REPLACE FUNCTION claim_anonymous_profile(
  p_claim_token VARCHAR(255),
  p_auth_user_id UUID,
  p_display_name VARCHAR(50), 
  p_email VARCHAR(255)
)
```

#### API Endpoint
- **Route**: `POST /api/profile/claim`
- **Authentication**: Required (email must be confirmed)
- **Validation**: Claim token format, display name requirements
- **Atomicity**: All operations within single transaction

#### Error Handling
- Invalid/already claimed tokens return 404
- Users with existing profiles return 409
- Failed migrations return 500 with rollback

## Data Preservation Guarantees

This pattern ensures **100% data continuity** by migrating:

1. **Profile Information**: All personal details, preferences, and settings
2. **User Preferences**: Terrain preferences, resort associations, equipment
3. **Activity History**: Complete record of user interactions and LLM usage
4. **Media Assets**: Profile images and uploaded content
5. **Privacy Settings**: Public/private profile preferences

## Security Considerations

- **Service Client**: Uses elevated permissions to bypass RLS during migration
- **Row Locking**: `FOR UPDATE` prevents concurrent claim attempts
- **Validation**: Email confirmation required before claiming
- **Token Security**: Claim tokens nullified after successful migration

## Benefits

1. **Seamless UX**: Users retain all their anonymous work and preferences
2. **RLS Compliance**: New profile works correctly with security policies  
3. **Data Integrity**: Atomic migration prevents partial state issues
4. **Performance**: Optimized with targeted indexes on claim tokens
5. **Scalability**: Pattern works regardless of anonymous data volume

## Implementation Checklist

- [ ] Database migration function created
- [ ] API endpoint with proper validation
- [ ] Frontend claim flow integration
- [ ] Error handling and user feedback
- [ ] Performance optimization (indexes)
- [ ] Security testing (concurrent claims, token validation)
- [ ] Migration testing with large datasets