# Slack Notifications Setup Guide

## Quick Setup Instructions

### 1. Create Slack Workspace & App

1. Go to [slack.com](https://slack.com) and create a free workspace (if you don't have one)
2. Visit [api.slack.com/apps](https://api.slack.com/apps)
3. Click "Create New App" → "From scratch"
4. Name it "QuiverRack Notifications"
5. Select your workspace

### 2. Create Incoming Webhook

1. In your app settings, go to "Incoming Webhooks"
2. Toggle "Activate Incoming Webhooks" to ON
3. Click "Add New Webhook to Workspace"
4. Select the channel (e.g., #new-users or create a new one)
5. Copy the Webhook URL (looks like: `https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXX`)

### 3. Add to Environment Variables

Add to your `.env.local`:
```bash
SLACK_WEBHOOK_URL=your-webhook-url-here
```

### 4. Install Slack Mobile App

1. Download Slack app on your phone
2. Sign in to your workspace
3. Enable notifications for the channel you selected

## Testing the Webhook

You can test your webhook with curl:
```bash
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Test notification from QuiverRack!"}' \
  YOUR_WEBHOOK_URL
```

## Message Format

The Edge Function sends rich formatted messages with:
- User type (Anonymous/Registered)
- Email and display name
- Signup timestamp
- Direct link to admin dashboard
- User statistics

## Customization

Edit the message format in:
- `/supabase/functions/notify-new-user/index.ts`

## Disable Notifications

To temporarily disable notifications:
1. Comment out the webhook URL in `.env.local`
2. Or disable the trigger in the database migration