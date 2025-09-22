# Production Environment Variables

This document lists all required environment variables for deploying QuiverRack to production.

## Required Environment Variables

### Supabase Configuration
```bash
# Your Supabase project URL
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co

# Your Supabase anonymous key (safe for client-side)
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key

# Your Supabase service role key (server-side only, keep secret!)
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
```

### Application Configuration
```bash
# Your production domain (must be HTTPS)
NEXT_PUBLIC_APP_URL=https://quiverrack.com

# Environment name
NEXT_PUBLIC_ENVIRONMENT=production
```

### OAuth Configuration (Future Use)
```bash
# Google OAuth
NEXT_PUBLIC_GOOGLE_CLIENT_ID=your-google-client-id

# Facebook OAuth  
NEXT_PUBLIC_FACEBOOK_APP_ID=your-facebook-app-id

# Apple OAuth
NEXT_PUBLIC_APPLE_CLIENT_ID=your-apple-client-id
```

### Email Service (Future Use)
```bash
# Resend API key for transactional emails
RESEND_API_KEY=your-resend-api-key
```

### Analytics & Monitoring
```bash
# Vercel Analytics ID
VERCEL_ANALYTICS_ID=your-vercel-analytics-id

# Sentry Error Monitoring (Required for production)
NEXT_PUBLIC_SENTRY_DSN=your-sentry-dsn
SENTRY_AUTH_TOKEN=your-sentry-auth-token
SENTRY_ORG=quiverrack
SENTRY_PROJECT=web-app
```

### Weather API (if using external service)
```bash
# OpenWeatherMap API key (if not using Open-Meteo)
OPENWEATHER_API_KEY=your-api-key
```

## Vercel Configuration

When deploying to Vercel, set these environment variables in your project settings:

1. Go to your Vercel project dashboard
2. Navigate to Settings → Environment Variables
3. Add each variable for the Production environment
4. Ensure sensitive keys (SUPABASE_SERVICE_ROLE_KEY) are marked as sensitive

## Security Notes

1. **Never commit these values to git**
2. **SUPABASE_SERVICE_ROLE_KEY** should only be available server-side
3. **Use different keys for production vs development**
4. **Rotate keys regularly** and update in Vercel

## Pre-deployment Checklist

- [ ] All required variables are set in Vercel
- [ ] NEXT_PUBLIC_APP_URL uses HTTPS and correct domain
- [ ] Supabase production instance is configured
- [ ] Service role key is marked as sensitive in Vercel
- [ ] OAuth redirect URLs are configured in provider dashboards
- [ ] Email service is configured and tested

## Validation Script

Run this script locally to validate your `.env.production` file:

```bash
# Check if all required variables are set
required_vars=(
  "NEXT_PUBLIC_SUPABASE_URL"
  "NEXT_PUBLIC_SUPABASE_ANON_KEY"
  "SUPABASE_SERVICE_ROLE_KEY"
  "NEXT_PUBLIC_APP_URL"
)

for var in "${required_vars[@]}"; do
  if [ -z "${!var}" ]; then
    echo "❌ Missing: $var"
  else
    echo "✅ Set: $var"
  fi
done
```

## Additional Configurations

### Content Security Policy
Update the CSP in `next.config.js` to include your production domains:
- Supabase storage domain
- OAuth provider domains
- Analytics domains

### CORS Configuration
If your API will be accessed from different domains, configure CORS headers appropriately.

### Rate Limiting
Rate limiting is configured in the application code and uses the database. Ensure your Supabase instance can handle the expected load.