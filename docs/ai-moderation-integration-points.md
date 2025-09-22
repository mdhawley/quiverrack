# AI Moderation Integration Points

This document provides a quick reference for developers implementing AI moderation. All integration points have been marked with comments in the code.

## Database Schema Ready

The database has been prepared with all necessary tables and fields:
- `moderation_jobs` - Job queue for background processing
- Extended `comment_reports` with AI fields (source, confidence_score, ai_analysis)
- Added `moderation_status` to comments
- Created system user for AI actions

## Code Integration Points

### 1. Comment Creation Hook
**File**: `/apps/web/src/app/api/v1/questions/[id]/comments/route.ts`
**Line**: ~285

Look for: `// AI MODERATION HOOK: Queue moderation job`

This is where you'll queue new comments for AI moderation. The code is already written but commented out.

### 2. Report Escalation Hook
**File**: `/apps/web/src/app/api/v1/comments/[id]/report/route.ts`
**Line**: ~128

Look for: `// AI MODERATION HOOK: Escalate severe reports`

This allows user reports to trigger immediate AI review for harassment/inappropriate content.

### 3. Admin Dashboard Integration
**File**: `/apps/web/src/app/dashboard/admin/reports/page.tsx`
**Line**: ~262

The admin interface already supports displaying AI-generated reports with confidence scores.

## Configuration

### Environment Variables
All AI moderation features are controlled by environment variables. See `/apps/web/src/config/ai-moderation.ts` for the full list.

Key variable: `NEXT_PUBLIC_AI_MODERATION_ENABLED=false` (master switch)

### Feature Flags
The system supports:
- Gradual rollout by percentage
- New users only mode
- Specific user inclusion/exclusion

## Implementation Steps

1. **Enable the feature**:
   - Set environment variables
   - Uncomment the code sections marked with "AI MODERATION HOOK"

2. **Create the background processor**:
   - See `/docs/ai-moderation-implementation.md` section "Step 3: Create Background Worker"
   - Set up cron job to process the `moderation_jobs` table

3. **Implement AI service integrations**:
   - See example in documentation for OpenAI, Perspective API, Azure
   - Add your chosen service to `/lib/moderation/services/`

4. **Test in shadow mode**:
   - Enable AI but don't take actions
   - Monitor the `moderation_audit_log` table
   - Adjust thresholds based on results

## Zero Performance Impact

When `NEXT_PUBLIC_AI_MODERATION_ENABLED=false`:
- No AI checks are performed
- No jobs are queued
- No additional database queries
- The `shouldModerateContent()` function returns false immediately

## Monitoring

Once enabled, monitor:
- `/api/v1/admin/reports` - Will show AI-generated reports
- `moderation_jobs` table - Job processing status
- `moderation_audit_log` table - All AI actions

## Support

For detailed implementation instructions, see:
- `/docs/ai-moderation-implementation.md` - Complete 600+ line guide
- `/apps/web/src/types/moderation.ts` - TypeScript interfaces
- `/apps/web/src/config/ai-moderation.ts` - Configuration options