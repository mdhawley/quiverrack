# Testing Real-Time Notifications for Question Answers

## Prerequisites

1. **Apply the migration**: Run the migration `062_realtime_notifications.sql` in your Supabase database:
   ```sql
   -- In Supabase SQL Editor, run the contents of:
   -- packages/database/supabase/migrations/062_realtime_notifications.sql
   ```

2. **Verify migration success**: Check that:
   - The `user_notifications` table has `metadata` and `link` columns
   - The notification type constraint includes `'question_answered'`
   - The trigger `trigger_notify_question_author` exists on `question_comments` table

## Testing Steps

### 1. Setup Test Users
1. Open two browser windows/tabs (preferably different browsers or incognito)
2. **Window 1**: Sign up/login as User A
3. **Window 2**: Sign up/login as User B

### 2. Create a Question
1. **In Window 1 (User A)**:
   - Navigate to Questions section
   - Create a new question with a clear title
   - Note the question URL

### 3. Test Real-Time Notifications
1. **In Window 2 (User B)**:
   - Navigate to User A's question
   - Post an answer (top-level comment)

2. **In Window 1 (User A)** - You should see immediately:
   - 🔔 A toast notification appears with:
     - Title: "New Answer"
     - Message: "[User B's name] answered your question: [Question title]"
     - A "View" button that navigates to the question
   - The notification bell icon shows a red badge with count
   - The notification appears in the dropdown

### 4. Verify Notification Behavior
- ✅ **Should trigger notification**: Top-level answers from other users
- ❌ **Should NOT trigger**: 
  - Your own answers to your questions
  - Replies to existing comments (nested comments)
  - Comments from anonymous users

### 5. Test Edge Cases
1. **Long question titles**: Create a question with a title > 50 characters
   - Notification should truncate with "..."
   
2. **Multiple answers**: Have several users answer the same question
   - Each should trigger a separate notification

3. **Mark as read**: Click on a notification
   - Should mark it as read and update the count

## Debugging

If notifications aren't working:

1. **Check browser console** for WebSocket connection errors
2. **Verify Supabase Realtime is enabled**:
   - Go to Supabase Dashboard > Database > Replication
   - Ensure `user_notifications` table is enabled for realtime
   
3. **Check trigger execution**:
   ```sql
   -- Check if notifications are being created
   SELECT * FROM user_notifications 
   WHERE type = 'question_answered' 
   ORDER BY created_at DESC;
   ```

4. **Test trigger manually**:
   ```sql
   -- Insert a test comment (replace UUIDs with real ones)
   INSERT INTO question_comments (
     question_id, 
     user_id, 
     content, 
     parent_comment_id
   ) VALUES (
     'question-uuid-here',
     'commenter-uuid-here',
     'Test answer',
     NULL
   );
   ```

## Common Issues

1. **"Maximum update depth exceeded" error**:
   - Fixed by using the `RealtimeNotificationProvider` component

2. **Notifications not appearing**:
   - Ensure migration was applied
   - Check that user is not anonymous
   - Verify WebSocket connection in Network tab

3. **Trigger errors**:
   - Check PostgreSQL logs for trigger execution errors
   - Ensure all referenced tables and columns exist