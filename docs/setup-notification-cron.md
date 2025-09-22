# Setting Up Notification Processing

## Automatic Processing Options

### Option 1: Supabase Database Webhooks (Recommended)
1. Go to your Supabase Dashboard
2. Navigate to Database → Webhooks
3. Create a new webhook:
   - Name: `new-user-notifications`
   - Table: `auth.users`
   - Events: INSERT
   - URL: `https://[YOUR-PROJECT-REF].supabase.co/functions/v1/process-signup-notifications`
   - Add header: `Authorization: Bearer [YOUR-SERVICE-ROLE-KEY]`

### Option 2: Cron Job with pg_cron
```sql
-- Run this in SQL Editor to set up automatic processing every minute
SELECT cron.schedule(
  'process-signup-notifications',
  '* * * * *', -- Every minute
  $$
  SELECT net.http_post(
    url := 'https://[YOUR-PROJECT-REF].supabase.co/functions/v1/process-signup-notifications',
    headers := jsonb_build_object(
      'Authorization', 'Bearer [YOUR-SERVICE-ROLE-KEY]',
      'Content-Type', 'application/json'
    ),
    body := '{}'::jsonb
  );
  $$
);
```

### Option 3: External Cron Service
Use a service like cron-job.org or EasyCron:
- URL: `https://[YOUR-PROJECT-REF].supabase.co/functions/v1/process-signup-notifications`
- Method: POST
- Headers: `Authorization: Bearer [YOUR-SERVICE-ROLE-KEY]`
- Schedule: Every 1-5 minutes

## Deploy Edge Functions

```bash
# Deploy the notification processor
supabase functions deploy process-signup-notifications

# Set the Slack webhook secret
supabase secrets set SLACK_WEBHOOK_URL=your-webhook-url-here

# Optional: Set admin URL for production
supabase secrets set NEXT_PUBLIC_ADMIN_URL=https://admin.quiverrack.com
```

## Testing

1. Run the test script:
```bash
npx tsx scripts/test-signup-notifications.ts
```

2. Or manually trigger the Edge Function:
```bash
curl -X POST https://[YOUR-PROJECT-REF].supabase.co/functions/v1/process-signup-notifications \
  -H "Authorization: Bearer [YOUR-SERVICE-ROLE-KEY]" \
  -H "Content-Type: application/json"
```

## Monitoring

Check the notification queue:
```sql
-- View unprocessed notifications
SELECT * FROM user_signup_notifications 
WHERE processed = false 
ORDER BY created_at DESC;

-- View recent processed notifications
SELECT * FROM user_signup_notifications 
WHERE processed = true 
ORDER BY processed_at DESC 
LIMIT 10;

-- Check for errors
SELECT * FROM user_signup_notifications 
WHERE error IS NOT NULL 
ORDER BY created_at DESC;
```

## Disable Notifications

To temporarily disable notifications:
1. Remove or comment out the SLACK_WEBHOOK_URL environment variable
2. Or disable the cron job/webhook in Supabase Dashboard