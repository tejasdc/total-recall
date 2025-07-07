# Database Schema Plan - Total Recall POC

## Overview
This document outlines the database schema for the spaced repetition system in Total Recall. This is a simplified POC version focusing on core functionality.

## Tables

### 1. knowledge_base
Tracks which PDFs each user is studying.

```sql
-- Simplified knowledge_base table
CREATE TABLE public.knowledge_base (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    pdf_id UUID NOT NULL REFERENCES public.pdf_documents(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    next_review_date TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(user_id, pdf_id)
);

-- Index for efficient due date queries
CREATE INDEX idx_knowledge_base_next_review ON public.knowledge_base(user_id, next_review_date);
```

### 2. space_repetition
Stores the review history for each knowledge base entry.

```sql
-- Simplified space_repetition table (review history)
CREATE TABLE public.space_repetition (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    knowledge_base_id UUID NOT NULL REFERENCES public.knowledge_base(id) ON DELETE CASCADE,
    review_date TIMESTAMPTZ DEFAULT NOW(),
    score INTEGER NOT NULL CHECK (score >= 1 AND score <= 4)
);

-- Index for efficient history queries
CREATE INDEX idx_space_repetition_kb_date ON public.space_repetition(knowledge_base_id, review_date DESC);
```

## Score Mapping
- 1 = AGAIN (Complete blackout, no recall)
- 2 = HARD (Recalled with serious difficulty)
- 3 = GOOD (Recalled with some hesitation)
- 4 = EASY (Perfect recall, no hesitation)

## Integration with ts-fsrs
- The application will use ts-fsrs library to calculate `next_review_date` based on review history
- FSRS algorithm state will be computed on-demand from the review history
- This keeps the database schema simple while leveraging the full power of FSRS

## Future Enhancements (Post-POC)
- [ ] Add FSRS state fields (stability, difficulty, etc.) to knowledge_base for performance
- [ ] Add review_ratings enum table for better documentation
- [ ] Implement Row Level Security (RLS) policies
- [ ] Add support for multiple PDFs per knowledge base
- [ ] Add flashcard-level tracking (currently tracks at PDF level)
- [ ] Add more detailed review metadata (time spent, etc.)

## Notes
- This schema assumes flashcards are stored in a separate table (managed by another team member)
- The unique constraint on (user_id, pdf_id) ensures one knowledge base entry per user per PDF
- All timestamps use TIMESTAMPTZ for proper timezone handling