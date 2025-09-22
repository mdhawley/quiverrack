# AI Moderation Implementation Guide

This guide explains how to implement AI-powered content moderation in QuiverRack. The system has been designed to have zero performance impact when disabled and can be gradually rolled out.

## Table of Contents
1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Implementation Steps](#implementation-steps)
5. [AI Service Integration](#ai-service-integration)
6. [Testing Strategy](#testing-strategy)
7. [Rollout Plan](#rollout-plan)
8. [Monitoring & Optimization](#monitoring--optimization)
9. [Troubleshooting](#troubleshooting)

## Overview

The AI moderation system can automatically detect and handle:
- Toxic or harmful content
- Spam and promotional content
- Personal information disclosure
- Hate speech and harassment
- Sexual content
- Violence and self-harm references

### Key Features
- **Multiple AI services** support (OpenAI, Google Perspective, Azure)
- **Async processing** to maintain performance
- **Gradual rollout** capabilities
- **Human review** workflow integration
- **Audit trail** for all actions
- **User feedback** and appeals

## Architecture

```
User submits comment
        ↓
[Sync Check (optional)]
        ↓
Save to database
        ↓
Queue moderation job ← [Skip if AI disabled]
        ↓
Show comment (or "pending")
        ↓
[Background Worker]
        ↓
AI Analysis
        ↓
Take action (flag/hide/allow)
        ↓
Notify admins if needed
```

### Database Schema

The system uses these key tables:
- `moderation_jobs` - Queue for async processing
- `comment_reports` - Extended to include AI-generated reports
- `moderation_config` - Thresholds and rules
- `moderation_audit_log` - Track all actions

## Prerequisites

1. **Environment Variables**
   ```env
   # Master switch
   NEXT_PUBLIC_AI_MODERATION_ENABLED=false
   
   # Service API keys
   OPENAI_API_KEY=your-key-here
   PERSPECTIVE_API_KEY=your-key-here
   AZURE_CONTENT_MODERATOR_KEY=your-key-here
   AZURE_CONTENT_MODERATOR_ENDPOINT=https://your-instance.cognitiveservices.azure.com/
   
   # Service toggles
   AI_MODERATION_OPENAI_ENABLED=true
   AI_MODERATION_PERSPECTIVE_ENABLED=false
   AI_MODERATION_AZURE_ENABLED=false
   
   # Thresholds
   AI_MODERATION_AUTO_HIDE_THRESHOLD=0.85
   AI_MODERATION_FLAG_THRESHOLD=0.70
   AI_MODERATION_WARNING_THRESHOLD=0.60
   
   # Behavior
   AI_MODERATION_PROCESS_MODE=async
   AI_MODERATION_BLOCK_ON_HIDE=false
   
   # Rollout
   AI_MODERATION_ROLLOUT_PERCENTAGE=0
   AI_MODERATION_NEW_USERS_ONLY=true
   ```

2. **Background Job Processor**
   - Set up a cron job or serverless function
   - Recommended: Run every 30 seconds
   - Process up to 50 jobs per run

## Implementation Steps

### Step 1: Enable AI Moderation

1. Set environment variables
2. Run database migration (already done):
   ```bash
   supabase db push
   ```

### Step 2: Add AI Check to Comment Creation

Update `/api/v1/questions/[id]/comments/route.ts`:

```typescript
import { shouldModerateContent, AI_MODERATION_CONFIG } from '@/config/ai-moderation';
import { queueModerationJob } from '@/lib/moderation/queue';

// In POST handler, after creating comment:
if (shouldModerateContent(user.id, user.created_at)) {
  // Queue for AI moderation
  await queueModerationJob({
    entity_type: 'comment',
    entity_id: comment.id,
    job_type: 'ai_moderation',
    priority: isNewUser ? 9 : 5,
    payload: {
      text: comment.content,
      context: {
        user_id: user.id,
        question_title: question.title,
        user_join_date: user.created_at
      }
    }
  });
  
  // If sync mode, wait for result
  if (AI_MODERATION_CONFIG.behavior.processMode === 'sync') {
    const result = await processCommentSync(comment);
    if (result.action === 'hide') {
      // Delete comment and return error
      await supabase.from('question_comments').delete().eq('id', comment.id);
      return NextResponse.json(
        { error: 'Content violates community guidelines' },
        { status: 400 }
      );
    }
  }
}
```

### Step 3: Create Background Worker

Create `/api/cron/process-moderation-jobs/route.ts`:

```typescript
import { createModerationProcessor } from '@/lib/moderation/processor';

export async function GET(request: Request) {
  // Verify cron secret
  const authHeader = request.headers.get('authorization');
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  const processor = createModerationProcessor();
  const results = await processor.processJobs(50); // Process up to 50 jobs
  
  return Response.json({
    processed: results.processed,
    failed: results.failed,
    skipped: results.skipped
  });
}
```

### Step 4: Implement Moderation Processor

Create `/lib/moderation/processor.ts`:

```typescript
import { OpenAIModerator } from './services/openai';
import { PerspectiveModerator } from './services/perspective';
import { determineAction } from '@/config/ai-moderation';

export class ModerationProcessor {
  async processJob(job: ModerationJob) {
    const { entity_type, entity_id, payload } = job;
    
    try {
      // Get AI analysis from enabled services
      const analyses = await this.runAnalyses(payload.text);
      
      // Combine scores from all services
      const combinedScores = this.combineScores(analyses);
      
      // Determine action
      const { action, reason } = determineAction(
        combinedScores, 
        new Date(payload.context.user_join_date)
      );
      
      // Take action
      await this.executeAction(entity_type, entity_id, action, reason, analyses);
      
      // Update job as completed
      await this.completeJob(job.id, { action, analyses });
      
    } catch (error) {
      await this.failJob(job.id, error.message);
    }
  }
  
  async executeAction(
    entityType: string, 
    entityId: string, 
    action: string, 
    reason: string,
    analyses: AIAnalysis[]
  ) {
    switch (action) {
      case 'hide':
        // Soft delete the comment
        await supabase.from('question_comments')
          .update({ 
            is_deleted: true,
            deleted_by: SYSTEM_USER_ID,
            deleted_reason: reason
          })
          .eq('id', entityId);
        
        // Create auto-report
        await supabase.from('comment_reports').insert({
          comment_id: entityId,
          reporter_id: SYSTEM_USER_ID,
          reason: 'ai_toxicity',
          details: reason,
          source: 'ai',
          confidence_score: analyses[0].confidence,
          ai_analysis: analyses
        });
        break;
        
      case 'flag':
        // Create report for admin review
        await supabase.from('comment_reports').insert({
          comment_id: entityId,
          reporter_id: SYSTEM_USER_ID,
          reason: 'ai_suspicious_pattern',
          details: reason,
          source: 'ai',
          confidence_score: analyses[0].confidence,
          ai_analysis: analyses
        });
        break;
    }
    
    // Log action
    await supabase.from('moderation_audit_log').insert({
      action_type: action === 'hide' ? 'hide' : 'flag',
      entity_type: entityType,
      entity_id: entityId,
      actor_type: 'ai',
      actor_id: SYSTEM_USER_ID,
      reason,
      metadata: { analyses }
    });
  }
}
```

### Step 5: Create AI Service Integrations

Example OpenAI integration (`/lib/moderation/services/openai.ts`):

```typescript
export class OpenAIModerator {
  async analyze(text: string): Promise<AIAnalysis> {
    const response = await fetch('https://api.openai.com/v1/moderations', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        input: text,
        model: 'text-moderation-latest'
      })
    });
    
    const data = await response.json();
    const result = data.results[0];
    
    return {
      service: 'openai_moderation',
      model_version: data.model,
      timestamp: new Date().toISOString(),
      scores: result.category_scores,
      categories: Object.keys(result.categories).filter(cat => result.categories[cat]),
      flagged: result.flagged,
      action_recommended: result.flagged ? 'flag' : 'none',
      confidence: Math.max(...Object.values(result.category_scores))
    };
  }
}
```

## AI Service Integration

### OpenAI Moderation API (Recommended - Free)

**Pros:**
- Free to use
- Fast response times
- Good accuracy
- Well-documented

**Setup:**
1. Get API key from OpenAI
2. Set `OPENAI_API_KEY` environment variable
3. Enable with `AI_MODERATION_OPENAI_ENABLED=true`

### Google Perspective API (Free)

**Pros:**
- Free with generous limits
- Specialized in toxicity detection
- Multiple language support

**Setup:**
1. Enable API in Google Cloud Console
2. Create API key
3. Set `PERSPECTIVE_API_KEY`
4. Enable with `AI_MODERATION_PERSPECTIVE_ENABLED=true`

### Azure Content Moderator

**Pros:**
- Image moderation support
- Custom blacklists
- More detailed categories

**Setup:**
1. Create Azure Cognitive Services resource
2. Get endpoint and key
3. Set environment variables
4. Enable with `AI_MODERATION_AZURE_ENABLED=true`

## Testing Strategy

### 1. Unit Tests

Test moderation logic without AI services:

```typescript
describe('AI Moderation', () => {
  it('should flag content above threshold', () => {
    const action = determineAction({ toxicity: 0.75 });
    expect(action.action).toBe('flag');
  });
  
  it('should use stricter thresholds for new users', () => {
    const newUserDate = new Date();
    const action = determineAction({ toxicity: 0.65 }, newUserDate);
    expect(action.action).toBe('flag');
  });
});
```

### 2. Integration Tests

Test with mock AI responses:

```typescript
it('should queue moderation job on comment creation', async () => {
  // Mock AI moderation as enabled
  process.env.NEXT_PUBLIC_AI_MODERATION_ENABLED = 'true';
  
  const response = await fetch('/api/v1/questions/123/comments', {
    method: 'POST',
    body: JSON.stringify({ content: 'Test comment' })
  });
  
  // Check job was queued
  const jobs = await supabase.from('moderation_jobs')
    .select('*')
    .eq('entity_type', 'comment');
    
  expect(jobs.data).toHaveLength(1);
});
```

### 3. Test Content Examples

Use these to verify AI behavior:

```typescript
const TEST_CONTENT = {
  // Should pass
  clean: "I love skiing at Vail, the powder was amazing!",
  
  // Should flag
  mildlyToxic: "This product is absolute garbage",
  
  // Should hide
  toxic: "[Insert clearly toxic content for testing]",
  
  // Edge cases
  sarcasm: "Oh great, another genius comment",
  context: "I'm going to kill it on the slopes!" // Should understand context
};
```

## Rollout Plan

### Phase 1: Shadow Mode (Week 1-2)
- Enable AI but don't take actions
- Log all decisions to `moderation_audit_log`
- Review false positives/negatives
- Adjust thresholds

```env
AI_MODERATION_ROLLOUT_PERCENTAGE=100
AI_MODERATION_PROCESS_MODE=async
# But don't actually hide/flag in processor
```

### Phase 2: New Users Only (Week 3-4)
- Enable for users < 7 days old
- Monitor impact on user experience
- Gather feedback

```env
AI_MODERATION_NEW_USERS_ONLY=true
AI_MODERATION_BLOCK_ON_HIDE=false
```

### Phase 3: Gradual Rollout (Week 5-6)
- Increase percentage gradually
- Monitor performance metrics
- Adjust thresholds based on data

```env
AI_MODERATION_ROLLOUT_PERCENTAGE=10  # Then 25, 50, 100
```

### Phase 4: Full Launch
- Enable for all users
- Consider sync mode for high-risk content
- Add user appeals process

## Monitoring & Optimization

### Key Metrics to Track

1. **Performance Metrics**
   - Job processing time
   - Queue depth
   - API response times
   - Failed job rate

2. **Accuracy Metrics**
   - False positive rate
   - False negative rate
   - Human override rate
   - Appeal success rate

3. **User Impact**
   - Comments blocked rate
   - User complaints
   - Engagement changes

### Dashboard Queries

```sql
-- Daily moderation stats
SELECT 
  DATE(created_at) as date,
  COUNT(*) as total_jobs,
  COUNT(*) FILTER (WHERE status = 'completed') as completed,
  COUNT(*) FILTER (WHERE result->>'action' = 'hide') as hidden,
  COUNT(*) FILTER (WHERE result->>'action' = 'flag') as flagged,
  AVG(processing_time_ms) as avg_time_ms
FROM moderation_jobs
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY DATE(created_at)
ORDER BY date DESC;

-- False positive tracking
SELECT 
  COUNT(*) as ai_hidden,
  COUNT(*) FILTER (WHERE human_verified AND status = 'dismissed') as false_positives,
  ROUND(100.0 * COUNT(*) FILTER (WHERE human_verified AND status = 'dismissed') / COUNT(*), 2) as false_positive_rate
FROM comment_reports
WHERE source = 'ai' 
AND created_at > NOW() - INTERVAL '30 days';
```

### Performance Optimization

1. **Caching**
   - Cache AI responses for identical text
   - Use Redis or in-memory cache
   - TTL: 1 hour for development, 24 hours for production

2. **Batching**
   - Process multiple comments in single AI request
   - Reduces API costs and latency

3. **Smart Scheduling**
   - Prioritize new user content
   - Deprioritize edits to already-approved content
   - Skip trusted users after threshold

## Troubleshooting

### Common Issues

1. **Jobs stuck in 'processing'**
   ```sql
   -- Reset stuck jobs
   UPDATE moderation_jobs 
   SET status = 'pending', attempts = attempts + 1
   WHERE status = 'processing' 
   AND started_at < NOW() - INTERVAL '5 minutes';
   ```

2. **High false positive rate**
   - Lower thresholds gradually
   - Add context to AI requests
   - Consider service-specific adjustments

3. **Slow processing**
   - Increase worker frequency
   - Add more workers
   - Check AI service latency

### Debug Mode

Enable detailed logging:

```env
AI_MODERATION_DEBUG=true
AI_MODERATION_LOG_SCORES=true
```

### Manual Testing

Test specific content:

```bash
curl -X POST http://localhost:3000/api/admin/test-moderation \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"text": "Test content here"}'
```

## Security Considerations

1. **API Key Security**
   - Never expose keys in client code
   - Use environment variables
   - Rotate keys regularly

2. **Rate Limiting**
   - Implement per-user limits
   - Queue overflow protection
   - Cost controls for paid APIs

3. **Privacy**
   - Don't log full content in production
   - Anonymize data for analysis
   - Comply with data retention policies

## Future Enhancements

1. **Machine Learning**
   - Train custom model on admin decisions
   - Improve accuracy over time
   - Reduce API costs

2. **Multi-modal Support**
   - Image moderation
   - Video content analysis
   - Audio transcription and checking

3. **Advanced Features**
   - Sentiment analysis
   - Contextual understanding
   - Language translation before checking

4. **User Features**
   - Pre-submission warnings
   - Suggested rewrites
   - Educational feedback