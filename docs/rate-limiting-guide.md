# Rate Limiting Guide

## Overview

QuiverRack implements a liberal multi-tier rate limiting system designed to prevent abuse while avoiding false positives for legitimate users, especially those on shared IPs (schools, businesses).

## Key Features

### 1. Multi-Tier Limits
- **Per Minute**: Quick burst protection
- **Per Hour**: Sustained usage monitoring  
- **Per Day**: Long-term usage patterns

### 2. Liberal Limits for Shared IPs
```
Unauthenticated Users:
- 100 requests/minute
- 3,000 requests/hour (~50/min sustained)
- 20,000 requests/day

Authenticated Users:
- 200 requests/minute
- 6,000 requests/hour
- 50,000 requests/day
```

### 3. Session-Based Tracking
- Anonymous sessions via secure cookies
- Browser fingerprinting for better identification
- 30-day session duration
- Reduces false positives from shared IPs

### 4. Pattern Detection
Detects suspicious behavior without blocking legitimate users:
- **Sequential ID scanning**: Accessing resources in numeric order
- **Rapid requests**: >5 requests/second triggers monitoring
- **Database enumeration**: Accessing many unique resources

### 5. Progressive Responses
Instead of hard blocks:
1. **Allow**: Normal access
2. **Throttle**: 1-5 second delays
3. **Challenge**: Headers indicate CAPTCHA needed
4. **Block**: Only for severe abuse

## Monitoring

### Database Tables
- `rate_limits`: Tracks request counts per window
- `anonymous_sessions`: Session tracking
- `request_patterns`: Detected suspicious patterns
- `audit_logs`: All security events

### Key Metrics to Monitor
1. **False Positive Rate**: Users incorrectly rate limited
2. **Pattern Detection Accuracy**: Legitimate vs actual scrapers
3. **Session Distribution**: Sessions per IP
4. **Endpoint Usage**: Which endpoints hit limits

### Queries for Monitoring

```sql
-- Top rate limited IPs
SELECT identifier, COUNT(*) as limit_hits
FROM audit_logs
WHERE action = 'rate_limit_exceeded'
AND created_at > NOW() - INTERVAL '1 day'
GROUP BY identifier
ORDER BY limit_hits DESC
LIMIT 20;

-- Suspicious patterns detected
SELECT pattern_type, COUNT(*) as detections
FROM request_patterns
WHERE detected_at > NOW() - INTERVAL '1 day'
GROUP BY pattern_type;

-- Sessions per IP
SELECT ip_address, COUNT(*) as session_count
FROM anonymous_sessions
WHERE last_seen > NOW() - INTERVAL '1 day'
GROUP BY ip_address
HAVING COUNT(*) > 5
ORDER BY session_count DESC;
```

## Adjusting Limits

### For Known Good Actors
Consider exempting:
- Educational institutions (by IP range)
- Partner organizations
- Internal services

### Endpoint-Specific Limits
Data-heavy endpoints have custom limits:
- `/api/v1/skis/browse`: 30/min, 1000/hr
- `/api/v1/profiles/search`: 50/min, 1500/hr

### Configuration
Edit `src/lib/multi-tier-rate-limiter.ts`:
```typescript
export const LIBERAL_UNAUTH_LIMITS: RateLimitConfig = {
  perMinute: 100,
  perHour: 3000,
  perDay: 20000,
};
```

## Troubleshooting

### User Reports Being Blocked
1. Check `audit_logs` for their identifier
2. Review `request_patterns` for false positives
3. Check if multiple users share same session
4. Consider increasing limits or exempting IP range

### High Database Load
1. Ensure cleanup job runs regularly
2. Check index usage with `EXPLAIN`
3. Consider archiving old data
4. Use memory cache in development

### Pattern Detection Too Aggressive
Adjust thresholds in `scraping-detector.ts`:
- Sequential ratio: 0.7 (70% sequential = suspicious)
- Rapid threshold: 5 req/sec
- Enumeration threshold: 100 unique resources

## Best Practices

1. **Monitor Weekly**: Review false positives and adjust
2. **Communicate Limits**: Add to API documentation
3. **Gradual Rollout**: Start liberal, tighten if needed
4. **Exempt Partners**: Whitelist known good actors
5. **User Feedback**: Provide clear error messages with support contact