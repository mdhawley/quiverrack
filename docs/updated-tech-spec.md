# QuiverRack Technical Specification v1.1

## Table of Contents
1. [Project Overview](#project-overview)
2. [User Stories](#user-stories)
3. [Technical Architecture](#technical-architecture)
4. [Authentication & User Management](#authentication--user-management)
5. [Data Models](#data-models)
6. [UI/UX Specifications](#uiux-specifications)
7. [API Endpoints](#api-endpoints)
8. [Privacy & Security](#privacy--security)
9. [Image Handling](#image-handling)
10. [Error Handling](#error-handling)
11. [Development Environment](#development-environment)
12. [MVP Scope Definition](#mvp-scope-definition)
13. [Development Timeline](#development-timeline)
14. [Future Enhancements](#future-enhancements)

## Project Overview

QuiverRack is a web application where skiers create resort-focused profiles to share with the skiing community. Users can showcase their abilities, preferred terrain, favorite resorts, and ski equipment collection.

### Technology Stack
- **Backend**: Supabase (PostgreSQL database + Edge Functions)
- **Frontend**: Next.js hosted on Vercel
- **CSS Framework**: Tailwind CSS
- **Authentication**: Supabase Auth with OAuth support (Google, Facebook, Apple)
- **Storage**: Supabase Storage for images
- **Analytics**: Vercel Analytics (basic page tracking for MVP)
- **Email Service**: Resend
- **Image Optimization**: Vercel Image Optimization
- **Content Moderation**: Not implemented in MVP

### Key Features
- Anonymous profile creation with later account claiming
- Comprehensive ski equipment database with cascading selection
- Mobile-first responsive design  
- Public profile sharing with privacy controls
- Social authentication (Google, Facebook, Apple) - all three in MVP
- Quiver photo showcase (photo section hidden if no image uploaded)
- API designed for future mobile app support

## User Stories

### Anonymous User Flow
1. **As a visitor**, I can create a skier profile without creating an account
2. **As an anonymous user**, I can fill out my skiing preferences and equipment
3. **As an anonymous user**, I can view my profile and receive a shareable link
4. **As an anonymous user**, I can return to my profile using browser cookies (persist for 1 week)
5. **As an anonymous user**, I am taken directly to my dashboard when returning (if cookie exists)

### Account Creation & Profile Claiming
1. **As an anonymous user**, I can create an account to claim my profile
2. **As a user with an existing account**, I see a side-by-side comparison and choose which profile to keep
3. **As an authenticated user**, I can use OAuth providers (Google, Facebook, Apple) - all three required for MVP
4. **As an authenticated user**, my display name becomes my permanent profile URL
5. **As a user choosing between profiles**, the unchosen profile is archived (not deleted)

### Profile Management
1. **As an authenticated user**, I can edit all profile fields except display name (never changeable)
2. **As a user**, I can toggle my profile between public and private
3. **As a user**, I can add skis from the database or manually enter custom skis
4. **As a user**, I can manage my resort preferences and equipment
5. **As a user**, I can set resort frequency (Regular/Occasional) via dropdown

## Technical Architecture

### Database Schema Overview
*See attached DATABASE_SCHEMA.md for complete details*

Key tables:
- `users` - Supabase Auth default profile table
- `ski_brands`, `ski_models`, `ski_model_years` - Equipment database (pre-seeded)
- `user_skis` - User's ski collection (max 30 skis per user)
- `resorts`, `user_resorts` - Resort information and preferences (596 US/Canada resorts pre-seeded)

### Anonymous to Authenticated Migration
*See attached anonymous-user-migration-prd.md for complete implementation*

Key approach:
- Create new profile with authenticated user ID
- Show side-by-side comparison of profiles
- Archive unchosen profile data
- Log which profile was chosen for analytics
- Maintain RLS compliance
- Atomic transaction for data integrity

### API Design
- Versioned endpoints: `/api/v1/`
- Rate limiting implemented from start
- Authentication tokens for future mobile app
- RESTful design with consistent JSON responses
- Proper error formatting for mobile compatibility

## Authentication & User Management

### Account Creation Flow

#### Required Fields
- Email (with verification required - enforced on account creation)
- Password (if not using OAuth)
- Display name (unique, permanent)

#### Display Name Requirements
- Must be unique across the platform
- 3-50 characters
- Alphanumeric + underscores only
- Cannot be changed after creation (ever)
- Variations allowed (john.doe and john_doe can coexist)
- Real-time availability checking
- No reserved names list for MVP

### Session Management
- Anonymous sessions persist for 1 week via browser cookies
- If anonymous profile exists, user goes directly to dashboard
- Only one anonymous profile per device (no multiple creation)
- Authenticated sessions follow Supabase defaults
- Session data preserved during anonymous → authenticated transition

### OAuth Providers (All required for MVP)
- Google
- Facebook  
- Apple

## Data Models

### Profile Data Structure

```typescript
interface UserProfile {
  // Core Information
  id: string;
  displayName: string;
  email: string;
  avatarUrl?: string;
  bio?: string; // max 500 characters, plain text only
  location?: string;
  
  // Skiing Information
  skierAbility: 'beginner' | 'intermediate' | 'advanced' | 'expert';
  skiingStyle?: string;
  terrainPreferences: TerrainType[];
  
  // Equipment & Resorts
  skis: UserSki[]; // max 30 skis
  resorts: UserResort[];
  
  // Metadata
  isPublic: boolean;
  createdAt: Date;
  updatedAt: Date;
  claimedAt?: Date;
  
  // Compliance
  termsAcceptedAt: Date;
  privacyAcceptedAt: Date;
  cookieConsentAt?: Date;
  analyticsOptOut: boolean;
}

interface TerrainType {
  type: 'Groomers' | 'Moguls' | 'Powder' | 'Park' | 'Trees' | 'Steeps' | 'Backcountry';
}

interface UserSki {
  id: string;
  brand: string;
  model: string;
  year: number;
  length: number; // cm only
  isVerified: boolean; // false for manual entries
  isManualEntry: boolean;
  notes?: string;
}

interface UserResort {
  id: string;
  resortName: string;
  frequency: 'regular' | 'occasional'; // required, defaults to 'regular'
}
```

## UI/UX Specifications

### Design System Summary
- **Color Palette**: "Powder Day" theme
  - Primary: #0D47A1 (Deep Sky Blue)
  - Secondary: #00897B (Teal)
  - Accent: #FF5252 (Safety Red)
  - Neutrals: #FFFFFF, #757575, #F5F5F5
- **Typography**: System fonts (-apple-system, BlinkMacSystemFont, etc.)
- **Border Radius**: 12-24px for cards, 25px for pills
- **Shadows**: Subtle (0 2px 20px rgba(0,0,0,0.06))
- **Animations**: fadeIn, fadeInUp, slideUp (0.8s ease)
- **Language**: English only for MVP

### Completed Page Designs
1. **Homepage** - Marketing landing page
2. **Onboarding Form** - Single page with all profile fields
3. **Ski Entry Modal** - Progressive disclosure for database entries
4. **Public Profile** - AllTrails-style with left sidebar
5. **Dashboard** - Same as public profile with edit controls

### Resort Search Implementation
- Hybrid approach:
  1. User types 2-3 characters
  2. Server returns matching resorts (~50-100 results)
  3. Further filtering happens client-side
- Autocomplete from 596 pre-seeded resorts
- International resorts prepared in database for future

### Resort Frequency
- Dropdown selection with two options: "Regular" and "Occasional"
- Required field, defaults to "Regular"
- No ability to have null value in UI

## API Endpoints

### Versioning
All endpoints prefixed with `/api/v1/`

### Authentication
- `POST /api/v1/auth/signup` - Create account
- `POST /api/v1/auth/signin` - Sign in
- `POST /api/v1/auth/signout` - Sign out
- `GET /api/v1/auth/session` - Get current session
- `POST /api/v1/auth/verify-email` - Verify email address
- `POST /api/v1/auth/resend-verification` - Resend verification email

### Profile Management
- `POST /api/v1/profile` - Create anonymous profile
- `GET /api/v1/profile/:id` - Get public profile
- `PUT /api/v1/profile/:id` - Update profile (authenticated)
- `POST /api/v1/profile/claim` - Claim anonymous profile
- `POST /api/v1/profile/compare` - Get side-by-side comparison

### Equipment
- `GET /api/v1/skis/brands` - List all brands
- `GET /api/v1/skis/models/:brandId` - Get models for brand
- `GET /api/v1/skis/years/:modelId` - Get years for model
- `GET /api/v1/skis/lengths/:modelId/:year` - Get lengths
- `POST /api/v1/user/skis` - Add ski to collection
- `DELETE /api/v1/user/skis/:id` - Remove ski

### Resorts
- `GET /api/v1/resorts` - List all resorts
- `GET /api/v1/resorts/search?q=` - Search resorts

## Privacy & Security

### Privacy Settings
- **Toggle**: Single "Private Profile" switch
- **Default**: Public
- **Private Mode**: Shows "This profile is private" to other users
- **Access**: Only profile owner can view private profiles

### Compliance & Legal

#### Cookie Consent
- Banner shown to all users (simplest approach)
- Accept-only option (no reject button for MVP)
- Stores preference in cookie

#### Required Agreements
1. **Terms of Service** - Required for all users (anonymous and authenticated)
2. **Privacy Policy** - Required for all users
3. **Age Verification** - Not implemented for MVP

#### Implementation
- **Anonymous Users**: Must accept ToS/Privacy before creating profile
- **Authenticated Users**: Must accept during signup
- **Email Notifications**: Verification email only for MVP

### Security Measures
- Email verification required on account creation
- Display name variations allowed (no complex reservation system)
- RLS policies on all user data
- HTTPS everywhere
- Secure session management
- Rate limiting on API endpoints

## Image Handling

### Avatar Images
- **Upload Size**: Any resolution
- **Storage Size**: Auto-resize to 200x200
- **Formats**: JPEG, PNG, WebP, HEIC (auto-convert HEIC client-side)
- **Max File Size**: 10MB

### Ski Quiver Photos
- **Upload Size**: Any resolution (handles modern phone cameras)
- **Processing**: Auto-resize to max 2000x2000 while maintaining aspect ratio
- **HEIC Conversion**: Client-side using libraries like `heic2any`
- **Generated Sizes**:
  - Thumbnail: 400x400 (cropped square)
  - Medium: 800x800 (fit within bounds)
  - Full: 2000x2000 (fit within bounds)
  - Original: Stored if under 25MB (for future use)
- **Max File Size**: 25MB
- **Display**: Hidden section if no photo uploaded

### Upload Experience
- **Error Handling**: 
  - Profile saves even if image upload fails
  - Warning shown to user
  - User can retry upload later
  - No retry limit for MVP
- **Progress indicator**: For large file uploads
- **Client-side compression**: Before upload
- **Orientation fix**: Auto-correct EXIF rotation issues

### Social Sharing (Open Graph)
- Basic implementation for MVP
- Generic QuiverRack branded image for all profiles
- Dynamic title: "John Doe | Expert Skier | QuiverRack"
- Standard meta tags for Facebook, Twitter, LinkedIn

## Error Handling

### Image Upload Failures
- Show warning message
- Allow profile save to continue
- Enable retry functionality
- No upload attempt limits for MVP

### Network Issues
- Retry logic for failed requests
- Offline detection with user notification
- Form data persistence in localStorage

### Validation Errors
- Inline field validation
- Clear error messages
- Display name availability checking

## Development Environment

### Setup
- Solo developer project
- Direct development on production Supabase
- Ski and resort databases pre-seeded
- Environment variables needed:
  - `NEXT_PUBLIC_SUPABASE_URL`
  - `NEXT_PUBLIC_SUPABASE_ANON_KEY`

### Future Considerations
- Second Supabase dev environment after user launch
- Staging environment post-MVP

## MVP Scope Definition

### Included in MVP ✅
- Anonymous profile creation with 1-week cookie persistence
- All 3 OAuth providers (Google, Facebook, Apple)
- Email verification (required)
- Basic analytics (page views via Vercel)
- Resort frequency (Regular/Occasional dropdown)
- Manual ski entry (flagged as unverified)
- Side-by-side profile comparison for claiming
- Basic Open Graph meta tags
- API v1 with rate limiting and auth tokens
- Client-side HEIC conversion
- Cookie consent banner for all users
- Profile archiving (not deletion)
- English language only

### Not in MVP ❌
- Admin interface (manual management)
- Profile deletion/recovery
- Age verification (under 13 blocking)
- Image content moderation
- Profile completion indicator
- Closed resort handling
- Advanced moderation tools
- Profanity filtering
- Reserved display names
- Email notifications beyond verification
- Rich text bio formatting
- Multiple language support

## Development Timeline

### Week 1-2: Core Anonymous Profile Flow
- Anonymous profile creation
- Cookie-based session management
- Basic profile form implementation
- Database connectivity

### Week 2-3: OAuth & Profile Claiming
- All 3 OAuth providers
- Email verification flow
- Profile claiming with comparison
- Profile archiving system

### Week 3-4: Ski Database Integration
- Cascading dropdown implementation
- Manual ski entry
- Ski management interface
- Resort search (hybrid approach)

### Week 4-5: Image Upload & Public Profiles
- Client-side HEIC conversion
- Image upload with error handling
- Public profile pages
- Basic Open Graph implementation

### Week 5-6: Polish & Deployment
- API versioning and rate limiting
- Cookie consent banner
- Testing and bug fixes
- Production deployment

## Future Enhancements

### Phase 2 Features (Post-MVP)
1. **Admin Interface**
   - Ski database management
   - User moderation tools
   - Analytics dashboard
   - Manual ski entry verification

2. **Profile Features**
   - Profile completeness indicator
   - Profile deletion/recovery
   - Age verification
   - Profanity filtering

3. **Enhanced Functionality**
   - Gear tracking (boots, bindings)
   - Trip logging
   - Social following
   - Equipment marketplace
   - Review system
   - International resort support
   - Multi-language support

4. **Advanced Features**
   - Content moderation for images
   - Advanced Open Graph with user photos
   - Email notifications for updates
   - Mobile app development

### Technical Debt Items
- Implement comprehensive analytics
- Add A/B testing framework
- Performance optimization
- Accessibility audit
- Create development/staging environments