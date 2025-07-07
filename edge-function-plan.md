# Edge Function Plan - Update Review Score (POC)

## Overview
Simple edge function to update review scores and calculate next review dates using ts-fsrs.

## Endpoint
```
POST /update-review-score
{
  "knowledge_base_id": "uuid",
  "score": 1-4,
  "review_date": "2024-01-01T10:00:00Z" // Optional, defaults to now
}
```

## Simplified Implementation (POC)

### Single File: `supabase/functions/update-review-score/index.ts`

```typescript
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'
import { fsrs, createEmptyCard, Rating } from 'https://esm.sh/ts-fsrs'

Deno.serve(async (req) => {
  // 1. Parse request body
  const { knowledge_base_id, score, review_date } = await req.json()
  
  // 2. Initialize FSRS and Supabase
  const f = fsrs()
  const supabase = createClient(...)
  
  // 3. For POC: Just create a new card each time
  const card = createEmptyCard(review_date ? new Date(review_date) : new Date())
  
  // 4. Apply the rating
  const scheduling_cards = f.repeat(card, review_date || new Date())
  const next_card = scheduling_cards[score].card
  
  // 5. Save review to database
  await supabase.from('space_repetition').insert({
    knowledge_base_id,
    score,
    review_date: review_date || new Date()
  })
  
  // 6. Update next review date
  await supabase.from('knowledge_base').update({
    next_review_date: next_card.due
  }).eq('id', knowledge_base_id)
  
  // 7. Return next review date
  return new Response(JSON.stringify({ 
    next_review_date: next_card.due 
  }))
})
```

## POC Simplifications
1. **No review history loading** - Each review starts fresh
2. **No complex state management** - FSRS handles it internally
3. **Minimal error handling** - Just focus on happy path
4. **No authentication** - Rely on Supabase's built-in auth
5. **Simple response** - Just return the next review date

## Database Operations
1. Insert into `space_repetition` table
2. Update `next_review_date` in `knowledge_base` table

## Dependencies
- `ts-fsrs` (via esm.sh)
- `@supabase/supabase-js` (via esm.sh)

## Notes
- The `review_date` parameter allows backdating reviews for POC testing
- Score mapping: 1=Again, 2=Hard, 3=Good, 4=Easy
- For production, would need to load review history and maintain FSRS state properly