# QuiverRack Testing Plan - User Types & Functionality

## Overview

This document provides a comprehensive testing plan for all user types in QuiverRack. The application supports three distinct user types, each with different capabilities and access levels.

## User Types

### 1. **Unauthenticated Users**
- No session, no profile
- Can browse public content
- Cannot create or save data
- Must create profile to access features

### 2. **Anonymous Users**
- Temporary session with profile
- 7-day data persistence
- Can use most features
- Can convert to authenticated account

### 3. **Authenticated Users**
- Full account with email/password
- Permanent data storage
- Access to all features
- Can sign in from any device

## Testing Scenarios by User Type

### 1. UNAUTHENTICATED USERS

#### Access & Navigation
- [ ] Can access homepage (/)
- [ ] Can browse public profiles (/profile/[username])
- [ ] Can view Browse Gear page
- [ ] Can view Browse Resorts page
- [ ] Can view Latest Questions (if feature enabled)
- [ ] Can access About, Privacy, Terms, Cookie Policy pages
- [ ] Cannot access any /dashboard/* routes (should redirect)

#### Actions Available
- [ ] Can start onboarding (/onboarding) → becomes anonymous
- [ ] Can sign up (/signup) → becomes authenticated
- [ ] Can log in (/login) → becomes authenticated
- [ ] Can search for public profiles
- [ ] Can view ski equipment details
- [ ] Can view resort information
- [ ] Can read questions and answers (no voting/commenting)

#### Header Navigation
- [ ] Shows "Login", "Sign Up", "Get Started" buttons
- [ ] No user dropdown menu
- [ ] No notification bell

### 2. ANONYMOUS USERS

#### Profile Creation & Management
- [ ] Can complete onboarding form
- [ ] Auto-generates display name "Skier[id]"
- [ ] Can add skis, resorts, terrain preferences, bio
- [ ] Can upload quiver photo
- [ ] Receives temporary profile URL (qvrck.com/p/[tempId])
- [ ] Profile data saved with 7-day expiration

#### Dashboard Access
- [ ] Can access all /dashboard/* routes
- [ ] Sees "Claim Profile" banner (AnonymousUserBanner)
- [ ] Can dismiss banner (persists in localStorage)
- [ ] Dashboard shows profile completion percentage
- [ ] Can view/edit all profile sections

#### Features Available
- [ ] **My Quiver**: Add/edit/remove skis (max 30)
- [ ] **My Resorts**: Add/remove favorite resorts with frequency
- [ ] **My Forecast**: View weather for saved resorts
- [ ] **Wishlist**: Add skis to wishlist
- [ ] **Profile**: Edit display name, bio, location, terrain preferences
- [ ] **Settings**: Toggle profile privacy (public/private)
- [ ] Cannot upload avatar (requires auth due to RLS policies)

#### Conversion Flow
- [ ] "Save Profile Permanently" button in user dropdown
- [ ] Can convert via /signup page
- [ ] Can convert from profile success page after onboarding
- [ ] After conversion:
  - [ ] Session updates without page refresh (fixed)
  - [ ] Banner disappears
  - [ ] All data migrated to authenticated account
  - [ ] Profile ID changes from temp to permanent

#### User Dropdown
- [ ] Shows "Anonymous User" or generated display name
- [ ] Has "Save Profile Permanently" option (amber highlight)
- [ ] Can sign out (clears localStorage)
- [ ] Has "Clear Local Data" option

### 3. AUTHENTICATED USERS

#### Full Profile Management
- [ ] All anonymous features PLUS:
- [ ] Can upload/change avatar
- [ ] Can receive email notifications
- [ ] Profile persists indefinitely
- [ ] Can sign in from any device
- [ ] Has permanent display name URL

#### Questions Feature (if enabled)
- [ ] Can ask questions
- [ ] Can answer questions
- [ ] Can vote on questions/answers
- [ ] Can favorite questions
- [ ] Can delete own questions/answers
- [ ] Receives notifications for responses

#### AI Assistant (if enabled)
- [ ] Can start AI conversations
- [ ] Can view conversation history
- [ ] Can provide feedback on AI responses
- [ ] Conversations are private to user

#### Enhanced Features
- [ ] **Notifications**: Bell icon with unread count
- [ ] **Profile URL**: Permanent custom URL (/profile/[displayName])
- [ ] **Account Settings**: Change email, password
- [ ] **Data Portability**: Can abandon account
- [ ] **Email Features**: Password reset, notifications

#### User Dropdown
- [ ] Shows display name and email
- [ ] No "Save Profile" option
- [ ] Full menu with all options
- [ ] Public profile link (if display name set)

## Critical User Flows to Test

### 1. Anonymous → Authenticated Conversion
1. Create anonymous profile via /onboarding
2. Add some data (skis, resorts, bio)
3. Click "Create Account" or "Save Profile Permanently"
4. Fill signup form with email, password, display name
5. Verify:
   - [ ] Session updates immediately (no refresh needed)
   - [ ] All data transferred correctly
   - [ ] Can log out and log back in
   - [ ] Data persists after logout
   - [ ] Anonymous banner gone
   - [ ] Avatar upload now works

### 2. Profile Privacy Settings
- [ ] Anonymous users can toggle privacy
- [ ] Private profiles return 404 via public URL
- [ ] Public profiles visible to all users
- [ ] Profile search only returns public profiles
- [ ] Dashboard always shows own profile

### 3. Data Persistence & Storage

#### Anonymous Users
- [ ] Data stored in PostgreSQL with expiry
- [ ] localStorage stores: profileId, profileCreated
- [ ] sessionStorage stores: onboardingFormData
- [ ] Cookies for session management
- [ ] Data expires after 7 days

#### Authenticated Users
- [ ] Data stored permanently in PostgreSQL
- [ ] Can clear browser data without losing profile
- [ ] Profile accessible from any device with login

### 4. Edge Cases to Test

#### Session Management
- [ ] Multiple anonymous profiles (different browsers)
- [ ] Expired anonymous session handling
- [ ] Browser back button after profile creation
- [ ] Clearing cookies mid-session
- [ ] Private/incognito mode behavior

#### Data Conflicts
- [ ] Duplicate display name during conversion
- [ ] Email already exists during signup
- [ ] Network errors during conversion
- [ ] Server errors during profile creation

#### Rate Limiting
- [ ] API endpoints have rate limits:
  - Auth endpoints: 10 requests/minute
  - Upload endpoints: 5 requests/minute
  - General API: 100 requests/minute

## API Endpoint Access Matrix

### Public Endpoints (all users)
```
GET  /api/v1/profiles/[identifier]    # Public profiles only
GET  /api/v1/profiles/recent          # Recent public profiles
GET  /api/v1/profiles/search          # Search public profiles
GET  /api/v1/skis/*                   # Ski database
GET  /api/v1/resorts/*                # Resort database
GET  /api/v1/questions/feed           # Read questions
GET  /api/v1/terrain-types            # Terrain options
```

### Anonymous + Authenticated
```
GET  /api/v1/profiles                 # Own profile
POST /api/v1/profiles                 # Create profile
PATCH /api/v1/profiles                # Update profile
POST /api/v1/upload/quiver           # Upload quiver photo
All  /api/v1/wishlist/*              # Wishlist operations
All  /api/v1/user/skis/*             # Manage skis
All  /api/v1/user/resorts/*          # Manage resorts
```

### Authenticated Only
```
POST /api/v1/upload/avatar           # Avatar upload
POST /api/v1/questions               # Create questions
POST /api/v1/questions/*/vote        # Vote on content
POST /api/v1/questions/*/comments    # Add comments
All  /api/v1/ai-conversations/*      # AI chat features
All  /api/v1/notifications/*         # Notifications
POST /api/v1/auth/change-password    # Account management
```

## Testing Tools & Utilities

### Development Tools

1. **Clear Local Data** (User Dropdown)
   - Clears all localStorage
   - Clears all sessionStorage  
   - Clears cookies for current domain
   - Reloads page to fresh state

2. **Reset Local Browser Data** (Admin Settings)
   - Same as above but from admin panel
   - Redirects to main app

3. **Reset Database** (Admin Settings)
   - Deletes ALL users and user data
   - Preserves seed data (resorts, skis, etc.)
   - Requires typing "RESET DATABASE"

### Browser Testing Setup

1. **Unauthenticated Testing**
   - Use incognito/private window
   - Or use "Clear Local Data" first

2. **Anonymous Testing**
   - Clear data, then go to /onboarding
   - Complete profile creation
   - Do NOT create account

3. **Authenticated Testing**
   - Create full account via /signup
   - Or convert anonymous profile

### Debugging Checklist

#### Check Browser DevTools
- [ ] **Application → Local Storage**
  - profileId
  - profileCreated
  - anonymousBannerDismissed
  - userPreferences

- [ ] **Application → Session Storage**
  - onboardingFormData

- [ ] **Application → Cookies**
  - Supabase auth cookies

- [ ] **Network Tab**
  - API response codes
  - Rate limit headers
  - Error responses

- [ ] **Console**
  - JavaScript errors
  - Failed API calls
  - Auth state changes

## Common Issues & Solutions

### Issue: UI shows wrong auth state after conversion
**Solution**: Fixed - session now set before redirect

### Issue: Can't upload avatar as anonymous
**Expected**: This is by design due to RLS policies
**Workaround**: Convert to authenticated account first

### Issue: Profile not found after creation
**Check**: 
- localStorage has profileId
- Profile privacy setting
- Using correct URL format

### Issue: Lost profile after clearing cookies
**Expected**: Anonymous profiles tied to browser
**Solution**: Create authenticated account for persistence

### Issue: "Display name taken" during conversion
**Solution**: Choose different display name
**Note**: Display names must be unique platform-wide

## Test Execution Checklist

### Daily Smoke Test
1. [ ] Create anonymous profile
2. [ ] Add ski and resort
3. [ ] Convert to authenticated
4. [ ] Verify data persisted
5. [ ] Log out and back in
6. [ ] Check public profile

### Weekly Full Test
- [ ] Run all unauthenticated user tests
- [ ] Run all anonymous user tests
- [ ] Run all authenticated user tests
- [ ] Test all edge cases
- [ ] Verify API access controls
- [ ] Check rate limiting
- [ ] Test on mobile devices

### Before Release
- [ ] Full test suite on staging
- [ ] Cross-browser testing (Chrome, Firefox, Safari)
- [ ] Mobile responsiveness
- [ ] Performance under load
- [ ] Security scan
- [ ] Accessibility audit

---

*Last Updated: [Current Date]*
*Version: 1.0*