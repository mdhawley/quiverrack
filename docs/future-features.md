# Future Features

This document outlines planned features for future development after MVP launch.

## Simple Messaging System

### Overview
A simple in-app messaging system to enable admin-to-user communication, particularly for gathering feedback and providing user support during early stages.

### Requirements
- Admin can send messages to individual users or groups
- Users receive notifications for new messages
- Persistent message center in user dashboard
- Read/unread status tracking
- Foundation for future user-to-user messaging

### Technical Architecture

#### Database Schema
```sql
-- Messages table
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  subject VARCHAR(255) NOT NULL,
  content TEXT NOT NULL,
  sender_type VARCHAR(50) DEFAULT 'admin',
  is_read BOOLEAN DEFAULT FALSE,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Notifications table
CREATE TABLE user_notifications (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  unread_count INTEGER DEFAULT 0,
  last_checked_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### API Endpoints
- `POST /api/admin/messages` - Send message from admin
- `GET /api/v1/messages` - Get user's messages
- `PATCH /api/v1/messages/:id/read` - Mark message as read
- `GET /api/v1/notifications/unread-count` - Get unread count

#### UI Components
- Notification bell icon in header navigation
- Unread count badge
- Messages page at `/dashboard/messages`
- Admin message compose interface

### Implementation Phases

#### Phase 1: Database & API (1-2 days)
- Create database tables with RLS policies
- Implement core API endpoints
- Add message sending to admin interface

#### Phase 2: User Interface (2 days)
- Add notification bell to navigation
- Create messages center page
- Integrate with existing dashboard

#### Phase 3: Admin Interface (1-2 days)
- Message compose modal
- Bulk messaging capabilities
- Message history and analytics

#### Phase 4: Enhancements (1-2 days)
- Real-time notifications using Supabase Realtime
- Message templates
- Advanced targeting options

### Estimated Effort
**Total: 5-8 days**

### Future Expansion
- User-to-user messaging
- Group messaging
- Message categories and threading
- Rich text support
- File attachments

### Priority
**Post-MVP** - Implement after core platform is stable and launched

---

## Other Future Features

### Advanced Analytics Dashboard
- User engagement metrics
- Ski data insights
- Popular brands and models
- Usage patterns

### Social Features
- User-to-user messaging (building on admin messaging)
- Friend connections
- Profile following
- Activity feeds

### Enhanced Ski Database
- Ski reviews and ratings
- Price tracking
- Retailer integration
- Ski recommendations

### Mobile App
- React Native application
- Push notifications
- Offline capabilities
- Camera integration for ski photos

### Advanced Search
- Full-text search across profiles
- Advanced filtering options
- Saved searches
- Search recommendations

### Content Management
- Blog/article system
- Video content integration
- User-generated content moderation
- SEO optimization

---

*Last updated: July 2025*