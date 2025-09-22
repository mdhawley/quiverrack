# Ski Link Processing System

## Overview

The ski link processing system automatically converts ski equipment mentions in AI responses into clickable links to the gear browse pages. This happens asynchronously to ensure fast response times while providing enhanced functionality.

## How It Works

### 1. **Immediate Response**
When an AI generates a response (either for questions or conversations), the response is:
- Saved to the database with `links_pending: true`
- Returned immediately to the user
- Shows a "Finding gear links..." indicator

### 2. **Background Processing**
After returning the response, the system:
- Creates a job in `ski_link_processing_jobs` table
- Triggers the processing API endpoint
- Extracts ski mentions using GPT-3.5-turbo
- Matches them to the database using fuzzy search
- Updates the content with markdown links

### 3. **Real-time Updates**
The UI components subscribe to database changes:
- Listen for updates to the content
- Replace the content when links are processed
- Hide the loading indicator

## Architecture

### Database Tables

- **ski_link_processing_jobs**: Tracks processing jobs
  - `id`: Job UUID
  - `comment_id` or `message_id`: Reference to content
  - `status`: pending, processing, completed, failed
  - `extracted_mentions`: JSON of extracted skis
  - `matched_links`: JSON of matched links

### API Endpoints

1. **POST /api/v1/extract-ski-mentions**
   - Uses OpenAI to extract ski mentions from text
   - Returns: `{brand, model, originalText, size}`

2. **POST /api/v1/ski-matcher**
   - Matches ski names to database entries
   - Uses fuzzy matching with aliases
   - Returns URLs and confidence scores

3. **POST /api/v1/process-ski-links**
   - Main processing endpoint
   - Orchestrates extraction and matching
   - Updates database with linked content

4. **GET /api/v1/process-ski-links/cron**
   - Processes pending jobs in batch
   - Can be called by monitoring services
   - Handles up to 10 jobs per invocation

### Database Functions

- `create_link_processing_job()`: Creates a new processing job
- `update_content_with_links()`: Updates content with processed links
- `match_ski_by_name_with_aliases()`: Fuzzy ski matching

## Configuration

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-...
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional
CRON_SECRET=your-secret-key  # For securing cron endpoint
```

### Database Migrations

Apply migration `051_add_async_link_processing.sql` to set up:
- New columns on comment/message tables
- Job tracking table
- Processing functions
- RLS policies

## Testing

Run the test script to verify the system:

```bash
cd apps/web
tsx scripts/test-ski-links.ts
```

This tests:
1. Ski extraction from text
2. Database matching
3. Full processing pipeline
4. Cron job endpoint

## Monitoring

### Check Pending Jobs

```sql
-- View pending jobs
SELECT id, status, created_at, error_message
FROM ski_link_processing_jobs
WHERE status = 'pending'
ORDER BY created_at DESC;

-- Check failed jobs
SELECT id, content, error_message, created_at
FROM ski_link_processing_jobs
WHERE status = 'failed'
ORDER BY created_at DESC;
```

### Manual Processing

If jobs are stuck, you can manually trigger processing:

```bash
# Process all pending jobs
curl http://localhost:3000/api/v1/process-ski-links/cron

# Process a specific job
curl -X POST http://localhost:3000/api/v1/process-ski-links \
  -H "Content-Type: application/json" \
  -d '{"job_id": "uuid-here"}'
```

## Deployment Considerations

### Option 1: Immediate Processing (Current)
- Jobs are processed immediately after creation
- Simple and reliable
- May add slight delay to response

### Option 2: Cron-based Processing
- Set up a cron job to call `/api/v1/process-ski-links/cron`
- Run every 1-5 minutes
- Better for high traffic

### Option 3: Supabase Edge Function
- Deploy the Edge Function in `/packages/supabase/functions/process-ski-links`
- Set up database webhook to trigger on job creation
- Most scalable option

## Troubleshooting

### Links Not Appearing

1. Check if job was created:
   ```sql
   SELECT * FROM ski_link_processing_jobs 
   WHERE comment_id = 'your-comment-id'
   OR message_id = 'your-message-id';
   ```

2. Check for errors in logs:
   - Look for `[requestId]` in server logs
   - Check `error_message` in job table

3. Verify API endpoints are accessible:
   - Test with the script: `tsx scripts/test-ski-links.ts`

### Performance Issues

- Increase job batch size in cron endpoint
- Implement caching for ski matches
- Use database indexes (already included in migration)

## Future Improvements

1. **Caching**: Cache ski name → URL mappings
2. **Batch Processing**: Process multiple contents in one LLM call
3. **Webhooks**: Use Supabase webhooks for instant processing
4. **Analytics**: Track which skis are mentioned most
5. **User Preferences**: Allow users to disable link processing