# AI-Powered Question Answering

## Overview

The QuiverRack platform now includes AI-powered answers for community questions. When users post questions about skiing, gear, resorts, or technique, they can request an AI-generated answer that takes into account their personal profile, equipment, and preferences.

## Key Features

### 1. Contextual AI Answers
- AI answers are personalized based on the user's:
  - Ski equipment (brands, models, lengths)
  - Favorite resorts and frequency
  - Location, height, and weight
  - Skill level
  - Terrain preferences

### 2. Hybrid Community + AI Approach
- Each question can have both AI and community answers
- AI answers are clearly marked with a robot emoji (🤖) and purple styling
- Community members can still provide human answers alongside AI responses
- Question owners can accept either AI or human answers as the best solution

### 3. Visual Differentiation
- AI answers have a gradient purple-blue background
- Display "QuiverRack AI Assistant" as the author
- Show which AI model was used (e.g., gpt-3.5-turbo)
- Include a note when the answer was personalized based on user profile

## Technical Implementation

### Database Schema

Added to `question_comments` table:
```sql
is_ai_generated BOOLEAN DEFAULT false
ai_metadata JSONB
```

The `ai_metadata` field stores:
- `model`: AI model used
- `usage`: Token usage information
- `cost`: Cost of generation
- `generated_at`: Timestamp
- `user_context_included`: Whether user profile was used
- `profile_data`: Summary of what profile data was available

### API Endpoints

**Generate AI Answer**
```
POST /api/v1/questions/[id]/ai-answer
Authorization: Bearer <token>
```

This endpoint:
1. Checks user authentication and feature flags
2. Retrieves the question details
3. Fetches user context (gear, resorts, preferences)
4. Generates a personalized AI response
5. Stores it as a special comment with `is_ai_generated = true`

### Configuration

1. Copy `.env.local.example` to `.env.local`
2. Set your AI provider and API key:
   ```
   AI_PROVIDER=openai
   OPENAI_API_KEY=sk-...
   ```

Supported providers:
- `openai` - OpenAI GPT models
- `anthropic` - Claude models
- `google` - Gemini models
- `mock` - For development/testing

### Feature Flags

The AI answer feature respects two feature flags:
- `FEATURE_QUESTIONS` - Must be enabled to see questions
- `FEATURE_AI_ASSISTANT` - Must be enabled to generate AI answers

## User Experience

### For Question Askers
1. Post a question as normal
2. Click "Get AI Answer" button to request AI assistance
3. AI generates a personalized answer within seconds
4. Can still wait for community answers
5. Can accept either AI or human answers as the solution

### For Other Users
- Can see AI answers alongside human answers
- Can upvote/downvote AI answers
- Can provide their own human answers
- AI answers are clearly marked to avoid confusion

## Safety and Limitations

### Rate Limiting
- Maximum 3 AI answers per question (configurable)
- Uses existing rate limiting for API requests
- Costs tracked per request

### Content Guidelines
- AI provides skiing-focused advice only
- Emphasizes safety in all recommendations
- Suggests professional instruction when appropriate
- Includes disclaimers for equipment recommendations

### Privacy
- User profile data is used only for personalization
- AI metadata is stored but doesn't include PII
- Users can see when their profile influenced the answer

## Future Enhancements

1. **Answer Quality Feedback**
   - Allow users to rate AI answer helpfulness
   - Use feedback to improve prompts

2. **Answer Regeneration**
   - Allow question owner to regenerate with different context
   - Try different AI models for comparison

3. **Smart Follow-ups**
   - AI suggests follow-up questions
   - Creates question threads for complex topics

4. **Answer Summarization**
   - AI summarizes multiple human answers
   - Highlights consensus and disagreements

## Development Notes

### Testing with Mock Provider
```bash
AI_PROVIDER=mock
MOCK_API_KEY=any-value
```

The mock provider returns contextual responses based on keywords without making real API calls.

### Monitoring
- Track AI answer generation in logs with request IDs
- Monitor token usage and costs
- Track user engagement with AI vs human answers