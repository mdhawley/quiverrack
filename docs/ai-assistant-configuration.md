# AI Assistant Configuration Guide

## Overview

The QuiverRack AI Assistant is a flexible, provider-agnostic chat system that helps users with skiing-related questions. It supports multiple LLM providers (OpenAI, Anthropic, Google, and a mock provider for testing) through a unified interface.

### Key Features
- 🔄 Provider abstraction - easily switch between AI providers
- 🎿 Ski-specific knowledge and context
- 👤 User profile personalization
- 💰 Cost tracking and limits
- 🔒 Rate limiting and security
- 💾 Conversation history storage
- 🧪 Mock provider for testing
- 🚦 Feature flag control for access management

## Configuration

### Environment Variables

Configure the AI Assistant by setting these environment variables in your `.env.local` file:

```bash
# Required: Choose your AI provider
AI_PROVIDER=openai|anthropic|google|mock

# Provider API Keys (only required for the selected provider)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_AI_API_KEY=...

# Optional: Override default model for the provider
AI_MODEL=gpt-4-turbo

# Optional: Rate limiting (defaults shown)
AI_RATE_LIMIT_REQUESTS_PER_HOUR=20
AI_RATE_LIMIT_TOKENS_PER_DAY=50000

# Optional: Cost controls (in USD)
AI_MAX_COST_PER_REQUEST=0.10
AI_MAX_COST_PER_USER_PER_MONTH=5.00
```

### Feature Flag Control

The AI Assistant is protected by a three-state feature flag (`FEATURE_AI_ASSISTANT`) that can be configured in the database:

- **`off`** - Feature is disabled for all users (default)
- **`invited-groups`** - Feature is enabled only for invited users
- **`public`** - Feature is enabled for all users

#### Setting Feature Flag State

Use the admin interface or directly update the database:

```sql
-- Enable for all users
UPDATE app_settings 
SET value = 'public' 
WHERE key = 'FEATURE_AI_ASSISTANT';

-- Enable for invited users only
UPDATE app_settings 
SET value = 'invited-groups' 
WHERE key = 'FEATURE_AI_ASSISTANT';

-- Disable completely
UPDATE app_settings 
SET value = 'off' 
WHERE key = 'FEATURE_AI_ASSISTANT';
```

#### Inviting Users to Test

To invite specific users when in `invited-groups` mode:

```sql
-- Invite a user to test AI Assistant
INSERT INTO invited_users (user_id, feature_key, invited_by, notes)
VALUES (
  '550e8400-e29b-41d4-a716-446655440000', -- User ID
  'FEATURE_AI_ASSISTANT',
  'admin-user-id', -- Optional: who sent invitation
  'Beta tester for AI Assistant feature'
);
```

#### Checking Feature Access

The system automatically handles feature flag checks:

- **API Routes**: All AI endpoints verify `isFeatureFlagEnabledForUser('FEATURE_AI_ASSISTANT', userId)`
- **UI Components**: `AIChatWidget` uses `useUserFeatureFlag('AI_ASSISTANT')` and hides when disabled
- **User Context**: Feature flags respect user invitations for `invited-groups` state

### Provider-Specific Configuration

#### OpenAI
```bash
AI_PROVIDER=openai
OPENAI_API_KEY=sk-...

# Available models:
# - gpt-4 (most capable, higher cost)
# - gpt-4-turbo (good balance)
# - gpt-3.5-turbo (fastest, lowest cost)
AI_MODEL=gpt-3.5-turbo
```

#### Anthropic
```bash
AI_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...

# Available models:
# - claude-3-opus-20240229 (most capable)
# - claude-3-sonnet-20240229 (balanced)
# - claude-3-haiku-20240307 (fastest)
AI_MODEL=claude-3-sonnet-20240229
```

#### Google Gemini
```bash
AI_PROVIDER=google
GOOGLE_AI_API_KEY=...

# Available models:
# - gemini-pro (general purpose)
# - gemini-pro-vision (with image support)
# - gemini-ultra (most capable)
AI_MODEL=gemini-pro
```

#### Mock Provider (Testing)
```bash
AI_PROVIDER=mock
# No API key required for mock provider
```

## API Reference

### Chat Endpoint

**POST** `/api/v1/ai/chat`

Send a message to the AI assistant.

#### Request Headers
```
Authorization: Bearer <user_token>
Content-Type: application/json
```

#### Request Body
```json
{
  "message": "What skis would you recommend for powder?",
  "conversationId": "optional-conversation-id",
  "context": [
    {
      "role": "user",
      "content": "Previous message"
    },
    {
      "role": "assistant", 
      "content": "Previous response"
    }
  ]
}
```

#### Response
```json
{
  "success": true,
  "data": {
    "content": "For powder skiing, I recommend...",
    "conversationId": "conv_123",
    "usage": {
      "promptTokens": 125,
      "completionTokens": 230,
      "totalTokens": 355
    },
    "model": "gpt-3.5-turbo"
  }
}
```

#### Error Response (Feature Disabled)
```json
{
  "error": "AI Assistant feature is currently not available",
  "status": 403
}
```

### Conversations List

**GET** `/api/v1/ai/conversations?limit=20&offset=0`

Retrieve user's chat history.

#### Response
```json
{
  "success": true,
  "data": [
    {
      "date": "2024-01-15",
      "messages": [...],
      "totalTokens": 1250,
      "totalCost": 0.0125
    }
  ],
  "pagination": {
    "total": 45,
    "limit": 20,
    "offset": 0,
    "hasMore": true
  }
}
```

### Get Conversation

**GET** `/api/v1/ai/conversation/{id}`

Retrieve a specific conversation.

### Delete Conversation

**DELETE** `/api/v1/ai/conversation/{id}`

Delete a conversation from history.

## Usage Guide

### Adding AI Chat to Your Page

1. **Floating Chat Widget** (Recommended)
```tsx
import { AIChatWidget } from '@/components/ai-assistant';

export default function MyPage() {
  return (
    <>
      {/* Your page content */}
      <AIChatWidget />
    </>
  );
}
```

2. **Embedded Chat Interface**
```tsx
import { AIChatInterface } from '@/components/ai-assistant';

export default function ChatPage() {
  return (
    <div className="h-[600px]">
      <AIChatInterface />
    </div>
  );
}
```

3. **With Question Suggestions**
```tsx
import { AIQuestionSuggestions } from '@/components/ai-assistant';

function SkiGearPage() {
  const [showChat, setShowChat] = useState(false);
  
  return (
    <>
      <AIQuestionSuggestions 
        context="gear"
        onSelectQuestion={(question) => {
          // Open chat with pre-filled question
          setShowChat(true);
        }}
      />
    </>
  );
}
```

### Context-Aware Suggestions

The AI assistant can provide context-specific suggestions based on the current page:

- `profile` - User profile page suggestions
- `gear` - Equipment-related suggestions
- `resort` - Resort-specific suggestions
- `general` - General skiing suggestions

## Rate Limiting & Costs

### Default Limits
- **Requests**: 20 per hour per user
- **Tokens**: 50,000 per day per user
- **Cost**: $0.10 max per request

### Token Usage by Model
| Provider | Model | Input $/1K | Output $/1K |
|----------|-------|------------|-------------|
| OpenAI | GPT-4 | $0.03 | $0.06 |
| OpenAI | GPT-3.5 | $0.0005 | $0.0015 |
| Anthropic | Claude-3-Opus | $0.015 | $0.075 |
| Anthropic | Claude-3-Sonnet | $0.003 | $0.015 |
| Google | Gemini Pro | $0.0005 | $0.0015 |

### Monitoring Usage

Check user's token usage:
```sql
SELECT 
  DATE(created_at) as date,
  SUM((llm_response->>'totalTokens')::int) as tokens_used,
  SUM(llm_cost_cents) / 100.0 as cost_usd
FROM user_inputs
WHERE user_id = ? 
  AND input_type = 'chat'
  AND created_at >= CURRENT_DATE
GROUP BY DATE(created_at);
```

## Security Considerations

### Authentication
- All AI endpoints require Bearer token authentication
- Tokens are validated against Supabase Auth
- Conversations are user-scoped

### Data Privacy
- No PII should be included in system prompts
- User profile data is used for personalization only
- Conversations are stored encrypted in the database
- Users can delete their conversation history

### Rate Limiting Implementation
```typescript
// Implemented in /api/v1/ai/chat/route.ts
const RATE_LIMIT_WINDOW = 60 * 60 * 1000; // 1 hour
const RATE_LIMIT_MAX_REQUESTS = 20;
const RATE_LIMIT_MAX_TOKENS_PER_DAY = 50000;
```

## Development & Testing

### Using Mock Provider

For development and testing without API costs:

```bash
# .env.local
AI_PROVIDER=mock
```

The mock provider returns contextual responses based on keywords:
- "ski recommendation" → Equipment suggestions
- "resort" → Resort recommendations  
- "parallel turn" → Technique advice
- "powder" → Powder skiing tips

### Testing Strategies

1. **Unit Tests**
```typescript
import { MockProvider } from '@quiverrack/shared/llm/providers';

const provider = new MockProvider({ 
  apiKey: 'mock',
  responseDelay: 0 
});

const response = await provider.chat([
  { role: 'user', content: 'ski recommendation' }
]);
```

2. **Integration Tests**
```typescript
// Test with mock provider
process.env.AI_PROVIDER = 'mock';

const response = await fetch('/api/v1/ai/chat', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ message: 'test' })
});
```

### Local Development Setup

1. Copy environment template:
```bash
cp .env.example .env.local
```

2. Set mock provider for development:
```bash
AI_PROVIDER=mock
```

3. Start development server:
```bash
pnpm dev
```

4. Apply database migration for feature flag:
```bash
# Navigate to database package
cd packages/database

# Apply the AI Assistant feature flag migration
pnpm db:migrate
```

## Advanced Topics

### Adding a New Provider

1. Create provider class implementing `LLMProvider`:
```typescript
// packages/shared/src/llm/providers/custom.ts
import { BaseLLMProvider } from './base';

export class CustomProvider extends BaseLLMProvider {
  get name(): string {
    return 'Custom';
  }
  
  get models(): string[] {
    return ['custom-model-1', 'custom-model-2'];
  }
  
  async chat(messages, options) {
    // Implement API call
  }
  
  estimateCost(promptTokens, completionTokens, model) {
    // Calculate costs
  }
}
```

2. Add to provider factory:
```typescript
// packages/shared/src/services/ai-assistant.ts
case 'custom':
  return new CustomProvider(providerConfig);
```

### Customizing System Prompts

Override the default system prompt:

```typescript
const aiService = new AIAssistantService({
  provider: 'openai',
  systemPrompt: `You are a specialized ski instructor AI...`
});
```

### Performance Optimization

1. **Response Caching** (future enhancement)
```typescript
// Cache common questions
const cachedResponse = await redis.get(`ai:${hash(message)}`);
if (cachedResponse) return cachedResponse;
```

2. **Token Optimization**
- Limit context to last 10 messages
- Summarize long conversations
- Use smaller models for simple queries

3. **Streaming Responses** (future enhancement)
```typescript
// Support for streaming responses
const stream = await aiService.streamChat(message);
for await (const chunk of stream) {
  // Send chunk to client
}
```

## Troubleshooting

### Common Issues

1. **"Rate limit exceeded"**
   - Check user's request count in last hour
   - Increase `AI_RATE_LIMIT_REQUESTS_PER_HOUR` if needed

2. **"Authentication failed"**
   - Verify API key is correct
   - Check provider service status

3. **"Daily token limit exceeded"**
   - User has hit 50k token limit
   - Consider increasing limit for power users

4. **High costs**
   - Switch to cheaper model (e.g., gpt-3.5-turbo)
   - Implement response caching
   - Add stricter per-request limits

### Debug Mode

Enable debug logging:
```typescript
// In API routes
console.log(`[${requestId}] AI request:`, { 
  provider, 
  model, 
  tokenCount 
});
```

### Provider-Specific Errors

**OpenAI**
- 429: Rate limit - implement exponential backoff
- 401: Invalid API key
- 500: OpenAI service issues

**Anthropic**
- Message formatting errors - ensure alternating user/assistant
- Context length limits - trim old messages

**Google**
- Safety filtering - adjust safety settings
- Region restrictions - use VPN or different region

## Support

For issues or questions:
1. Check this documentation
2. Review error logs in Supabase
3. Test with mock provider
4. Open an issue with details and logs