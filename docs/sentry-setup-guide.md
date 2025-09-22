# Sentry Setup Guide

This guide explains how to set up Sentry error monitoring for QuiverRack.

## Prerequisites

1. Create a Sentry account at https://sentry.io
2. Create a new project for QuiverRack (type: Next.js)
3. Note your DSN and create an auth token

## Installation

Sentry has already been added to the project dependencies. Run:

```bash
pnpm install
```

## Configuration Files

The following files have been created for Sentry integration:

### 1. `instrumentation.ts`
Handles both server and edge runtime initialization per Next.js 15 requirements.

### 2. `sentry.client.config.ts`
Client-side configuration with:
- Session replay (10% sample rate)
- Performance monitoring
- Error filtering

### 3. `src/app/global-error.tsx`
Catches and reports React rendering errors to Sentry.

### 4. `src/app/error.tsx`
Page-level error boundary with Sentry reporting.

### 5. `.sentryclirc`
Configuration for source map uploads during build.

## Environment Variables

Add these to your `.env.local` for development and Vercel for production:

```bash
# Client-side DSN (public)
NEXT_PUBLIC_SENTRY_DSN=https://your-key@sentry.io/your-project-id

# Server-side auth token (for source maps)
SENTRY_AUTH_TOKEN=sntrys_your_auth_token_here

# Optional - will use defaults from .sentryclirc
SENTRY_ORG=quiverrack
SENTRY_PROJECT=web-app
```

## Getting Your Sentry Credentials

### 1. DSN (Data Source Name)
1. Go to Settings → Projects → Your Project
2. Click on "Client Keys (DSN)"
3. Copy the DSN URL

### 2. Auth Token
1. Go to Settings → Auth Tokens
2. Create a new token with these scopes:
   - `project:releases`
   - `org:read`
   - `project:write`
3. Copy the token (starts with `sntrys_`)

## Features Configured

### Error Tracking
- Automatic error capture
- User-friendly error pages
- Stack trace collection
- User context (anonymized)

### Performance Monitoring
- 10% sample rate in production
- 100% sample rate in development
- Transaction tracing

### Session Replay
- 10% of normal sessions recorded
- 100% of sessions with errors recorded
- Helps debug user issues

### Filtering
- Ignores non-error events in development
- Filters out expected network errors
- Sanitizes sensitive data (cookies, PII)

## Testing Sentry

To test that Sentry is working:

1. Add a test button to trigger an error:

```tsx
<button onClick={() => {
  throw new Error('Test Sentry Error');
}}>
  Test Sentry
</button>
```

2. Check your Sentry dashboard for the error

## Build Configuration

During build, Sentry will:
1. Upload source maps for better error context
2. Create a release with commit SHA
3. Associate errors with releases

To disable source map uploads (e.g., for faster local builds):

```bash
SENTRY_UPLOAD_DISABLED=1 pnpm build
```

## Monitoring in Production

### Alerts
Set up alerts in Sentry for:
- Error rate spikes
- New error types
- Performance degradation
- Failed transactions

### Dashboard
Create a custom dashboard to monitor:
- Error volume
- User impact
- Performance metrics
- Most common errors

## Best Practices

1. **Don't log sensitive data**
   - User passwords
   - API keys
   - Personal information

2. **Use context for debugging**
   ```ts
   Sentry.setContext('subscription', {
     plan: user.plan,
     status: user.status
   });
   ```

3. **Add breadcrumbs for user actions**
   ```ts
   Sentry.addBreadcrumb({
     message: 'User clicked save',
     category: 'ui',
     level: 'info'
   });
   ```

4. **Group similar errors**
   ```ts
   Sentry.withScope(scope => {
     scope.setFingerprint(['api-error', endpoint]);
     Sentry.captureException(error);
   });
   ```

## Troubleshooting

### Errors not appearing in Sentry
1. Check DSN is correct
2. Verify environment variables are set
3. Check browser console for Sentry errors
4. Ensure not filtered by beforeSend

### Source maps not working
1. Verify SENTRY_AUTH_TOKEN is set
2. Check build logs for upload errors
3. Ensure release name matches

### Performance issues
1. Reduce tracesSampleRate if needed
2. Disable session replay if impacting performance
3. Review beforeSend filter efficiency