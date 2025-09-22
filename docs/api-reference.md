# API Reference

Base URL: `/api/v1`

## Profile Endpoints

### Get User Profile
Retrieve a user's complete profile data for the dashboard.

```
GET /profiles?profileId={userId}
```

**Query Parameters:**
- `profileId` (required): User's UUID

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "display_name": "string",
    "email": "string",
    "location": "string",
    "bio": "string",
    "avatar_url": "string",
    "skier_ability": "beginner|intermediate|advanced|expert",
    "terrain_preferences": ["Powder", "Trees"],
    "is_public": true,
    "user_resorts": [...],
    "user_skis": [...],
    "created_at": "timestamp",
    "updated_at": "timestamp"
  }
}
```

### Get Public Profile
Retrieve a public profile by display name or user ID.

```
GET /profiles/[identifier]
```

**URL Parameters:**
- `identifier`: Display name or UUID

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "displayName": "string",
    "bio": "string",
    "location": "string", 
    "avatar_url": "string",
    "skierAbility": "string",
    "terrainPreferences": ["string"],
    "isPublic": true,
    "skis": [...],
    "resorts": [...],
    "createdAt": "date",
    "updatedAt": "date"
  }
}
```

**Error Response (404):**
```json
{
  "error": "Profile not found or not public"
}
```

### Update Profile
Update user profile information.

```
PATCH /profiles?profileId={userId}
```

**Query Parameters:**
- `profileId` (required): User's UUID

**Request Body:**
```json
{
  "display_name": "string (optional)",
  "bio": "string (optional, max 500 chars)",
  "location": "string (optional, max 100 chars)",
  "avatar_url": "string (optional)",
  "skier_level": "beginner|intermediate|advanced|expert (optional)",
  "terrain_preferences": ["Powder", "Trees"] "(optional)",
  "is_public": true "(optional)"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    // Updated profile object
  }
}
```

**Notes:**
- All fields are optional
- Terrain preferences are handled as complete replacement
- Bio is validated for control characters

### Create Profile
Create a new anonymous user profile.

```
POST /profiles
```

**Request Body:**
```json
{
  "skierAbility": "beginner|intermediate|advanced|expert",
  "terrainPreferences": ["Powder", "Trees"],
  "bio": "string (optional)",
  "isPublic": true,
  "resorts": [
    {
      "id": 1,
      "name": "Vail",
      "state": "CO",
      "country": "USA",
      "frequency": "often|sometimes|rarely"
    }
  ],
  "skis": [
    {
      "brand": "Volkl",
      "model": "Mantra",
      "year": 2024,
      "length": 184,
      "ski_model_id": "123"
    }
  ]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "profileId": "uuid",
    "profile": {...},
    "auth": {
      "email": "anonymous@quiverrack.com",
      "password": "string",
      "userId": "uuid"
    }
  }
}
```

## Upload Endpoints

### Upload Avatar
Upload a user avatar image.

```
POST /upload/avatar
```

**Request:** FormData
- `file`: Image file (JPEG, PNG, WebP)
- `userId`: User's UUID

**Response:**
```json
{
  "success": true,
  "data": {
    "url": "https://storage.url/avatar.jpg",
    "path": "userId/filename.jpg"
  }
}
```

**Error Response (400):**
```json
{
  "error": "Invalid file type. Only JPEG, PNG, and WebP images are allowed."
}
```

**Constraints:**
- Max file size: 5MB
- Allowed types: image/jpeg, image/png, image/webp

## Equipment Endpoints

### Get Ski Brands
List all ski brands in the database.

```
GET /skis/brands
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Volkl"
    }
  ]
}
```

### Get Ski Models
Get models for a specific brand.

```
GET /skis/brands/[brandId]/models
```

**URL Parameters:**
- `brandId`: Brand ID

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Mantra",
      "brand_id": 1
    }
  ]
}
```

### Get Model Years
Get available years and lengths for a model.

```
GET /skis/models/[modelId]/years
```

**URL Parameters:**
- `modelId`: Model ID

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "year": 2024,
      "ski_model_id": 1,
      "available_lengths": [177, 184, 191]
    }
  ]
}
```

## Resort Endpoints

### Search Resorts
Search for resorts by name.

```
GET /resorts/search?q={query}
```

**Query Parameters:**
- `q`: Search query (min 2 characters)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Vail",
      "state": "CO",
      "country": "USA"
    }
  ]
}
```

## User Equipment Endpoints

### Add User Ski
Add a ski to user's quiver.

```
POST /user/skis
```

**Request Body:**
```json
{
  "userId": "uuid",
  "skiModelId": 1,
  "year": 2024,
  "length": 184
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    // Ski details
  }
}
```

### Remove User Ski
Remove a ski from user's quiver.

```
DELETE /user/skis/[id]
```

**URL Parameters:**
- `id`: User ski ID

**Response:**
```json
{
  "success": true
}
```

### Add User Resort
Add a resort to user's favorites.

```
POST /user/resorts
```

**Request Body:**
```json
{
  "userId": "uuid",
  "resortId": 1,
  "frequency": "often|sometimes|rarely"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    // Resort details
  }
}
```

### Remove User Resort
Remove a resort from user's favorites.

```
DELETE /user/resorts/[id]
```

**URL Parameters:**
- `id`: User resort ID

**Response:**
```json
{
  "success": true
}
```

## Error Responses

All endpoints may return these error responses:

### 400 Bad Request
```json
{
  "error": "Error message",
  "details": "Additional details (optional)"
}
```

### 404 Not Found
```json
{
  "error": "Resource not found"
}
```

### 500 Internal Server Error
```json
{
  "error": "Internal server error",
  "details": "Error details (in development only)"
}
```

## Notes

- All timestamps are in ISO 8601 format
- UUIDs are used for all user and resource IDs
- Terrain preferences use string names in API but are stored as IDs
- Display names are unique and cannot be changed after creation
- Anonymous users are created with auto-generated credentials