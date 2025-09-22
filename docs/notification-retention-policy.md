# Notification Retention Policy

This document outlines the notification retention and cleanup policies for QuiverRack.

## Retention Periods

### Default Retention Settings
- **Auto-Archive**: 30 days after creation
- **Hard Delete**: 90 days after creation

### Notification Types
All notification types follow the same retention policy by default:
- `FEATURE_INVITED`: 90 days retention, 30 days auto-archive
- `FEATURE_REMOVED`: 90 days retention, 30 days auto-archive

## Cleanup Process

### Automatic Archiving
- Notifications older than 30 days are automatically archived
- Archived notifications are still visible to users but marked as archived
- Archiving helps organize the notification list

### Hard Deletion
- Notifications older than 90 days are permanently deleted from the database
- This helps maintain database performance and compliance with data retention policies

### User Actions
- Users can soft delete notifications at any time
- Soft deleted notifications are hidden from the user but retained for the full retention period
- This allows for potential recovery if needed

## Cleanup Execution

### Manual Cleanup
Administrators can trigger manual cleanup via the admin API:
```bash
POST /api/v1/admin/notifications/cleanup
Authorization: Bearer <ADMIN_API_KEY>
```

### Scheduled Cleanup
For production environments, set up a cron job or scheduled function to run cleanup:
- Recommended frequency: Daily
- Optimal time: During low-traffic hours (e.g., 2 AM UTC)

### Example Cron Configuration
```bash
# Run daily at 2 AM UTC
0 2 * * * curl -X POST -H "Authorization: Bearer $ADMIN_API_KEY" https://your-domain.com/api/v1/admin/notifications/cleanup
```

## Database Functions

### Core Functions
- `archive_old_notifications()`: Archives notifications based on template settings
- `cleanup_old_notifications()`: Hard deletes notifications past retention period
- `soft_delete_notification(id, user_id)`: Soft deletes a specific notification

### Indexes
- `idx_user_notifications_active`: Optimizes queries for active notifications
- `idx_user_notifications_cleanup`: Optimizes retention cleanup queries

## Configuration

### Template-Based Retention
Each notification template can have custom retention settings:
- `retention_days`: Days to keep before hard deletion
- `auto_archive_days`: Days before automatic archiving

### Environment Variables
- `ADMIN_API_KEY`: Required for manual cleanup endpoint access

## Monitoring

### Cleanup Results
The cleanup endpoint returns statistics:
```json
{
  "success": true,
  "message": "Notification cleanup completed successfully",
  "results": {
    "archived": 45,
    "deleted": 12
  }
}
```

### Logging
All cleanup operations are logged with:
- Number of notifications archived
- Number of notifications deleted
- Execution time and status

## Best Practices

1. **Regular Monitoring**: Check cleanup logs to ensure the process runs successfully
2. **Retention Adjustment**: Monitor user feedback and adjust retention periods if needed
3. **Performance**: Run cleanup during low-traffic periods to minimize database impact
4. **Backup**: Ensure database backups include notification data for compliance

## Compliance

This retention policy helps ensure:
- **Performance**: Prevents notification table from growing indefinitely
- **Privacy**: Automatically removes old personal notification data
- **Storage**: Manages database storage costs effectively
- **User Experience**: Keeps notification lists relevant and manageable