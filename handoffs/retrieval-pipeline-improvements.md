# Agent Handoff: Retrieval Pipeline Improvements

**Date:** 2026-03-18  
**Target codebase:** `/Users/openclaw/Desktop/Mercury/`  
**Key files:**
- `message-api-configs/utils/get-query.ts` (and `get-query-v2.ts`) — query generation
- `message-api-configs/processors/memory-processor-v2.ts` — retrieval logic
- relevant Supabase functions (time-window path)

---

## Two improvements to make

---

## Improvement 1: Query Generation — Assistant-Recall & Specificity

### Problem

For `single-session-assistant` questions, the user is asking "what did **you** (the assistant) say in a previous conversation?" The current query generation treats this the same as user-fact recall — it generates a semantic query about the topic, not about what the assistant specifically said. This causes the retrieval to pull topically-related memories instead of the exact session where the assistant said something specific.

### Concrete examples from Batch 0

**Case `1d4da289`:**
- Question: *"You mentioned that companies use two-factor authentication to enhance security. Can you remind me what kind of two-factor authentication methods you were referring to?"*
- Generated query: `"types of two-factor authentication methods"` / `"how companies use two-factor authentication for security"`
- What was retrieved: A generic memory about firewall + security best practices (mentions 2FA briefly alongside 6 other recommendations) from a completely different session
- What was needed: The specific session where the assistant discussed biometric auth and OTP as 2FA methods
- **Root cause:** Query is topic-scoped ("2FA methods") — it matched any memory that mentions 2FA, not the one where it was specifically discussed in depth

**Case `e48988bc`:**
- Question: *"You mentioned a company doing a great job with sustainability in supply chain. Can you remind me which one?"*
- Generated query: `"company doing great job with sustainability in supply chain"`
- What was retrieved: A Reebok sustainability memory (user asked about Reebok's eco collections in a different session — topically close but NOT the answer session)
- What was needed: The Patagonia supply chain session (`answer_ultrachat_174360`)
- **Root cause:** Query matched a topically similar memory (sustainability + company) instead of the specific assistant-recommended one. The Reebok memory was semantically closer because it had more keyword overlap, while the Patagonia memory was about supply chain specifically.

**Case `1c0ddc50`:**
- Question: *"Can you suggest some activities I can do during my commute to work?"*
- Generated query: `"commute activities"` / `"commute productivity"`
- What was retrieved: Two memories about bike commute planning (how to get to work by bike) — completely different topic from podcast/audiobook preferences during commute
- What was needed: Memory about user's preference for history podcasts/audiobooks during commute
- **Root cause:** "commute" keyword matched bike logistics memories. The query was too surface-level and missed the "activity *during* commute" intent.

### What to change

In `get-query.ts` / `get-query-v2.ts`, improve the query generation prompt to handle two cases better:

**1. Assistant-recall questions** (user says "you mentioned", "you recommended", "you said", "remind me what you told me"):
- Detect this pattern (it's already partially detected via `wait_for_memory_reason: "explicit-remind"`)
- When detected: generate a more specific `inference_query` that captures WHAT was said, not just the topic
- Example: instead of `"types of 2FA methods"` → generate `"assistant explained specific 2FA methods biometric OTP"`
- The key insight: add the *answer content* as a hint in the query, not just the topic. The user is describing what they remember being told — use that description as part of the query.

**2. Preference/activity questions with context:**
- When the question includes context about the mode/situation (e.g. "during my commute"), preserve that context in the query
- Example: `"commute activities"` misses → `"activities to do while commuting, podcast audiobook"` would work better
- The inference_query should explore the *user's past preferences in this context*, not just the topic

### Implementation note
Read the current query generation prompt carefully before editing. The change should be additive — add guidance for these two patterns without breaking existing behavior. Do not change the output schema.

---

## Improvement 2: Time-Window Retrieval — Semantic Search After Filtering

### Problem

When `time_retrieval_mode = "all"` and a `time_frame` is set, the current logic:
1. Fetches all memories within the time window
2. Ranks by `relevance_score + time_proximity` combined
3. Returns top-K (max 5)

The problem: step 2 uses time proximity as part of the ranking score, which means **recency beats relevance**. Inside a time window, the most recently-created memories float to the top regardless of whether they're topically relevant.

### Concrete examples from Batch 0

**Case `75832dbd`:**
- Question: *"Can you recommend some recent publications or conferences that I might find interesting?"*
- Time frame detected: 7 days
- Status message: `"Time-focused ranking applied (relevance+time proximity). Kept 5/80 own memories"`
- What was retrieved: Motivation for social events, Yoga/Sarah, Edtech platform, Intersectional feminism, Komeda's breakfast — all from the 7-day window but **zero relevance** to publications/conferences/AI healthcare
- What was needed: Memory about user's interest in AI healthcare research and deep learning for medical imaging
- **Root cause:** 80 memories in the 7-day window, top-5 by time proximity — the recency score dominated, not semantic relevance

**Case `bc149d6b`:**
- Question: *"What is the total weight of the new feed I purchased in the past two months?"*
- Time frame detected: 2 months
- Status message: `"Time-focused ranking applied (relevance+time proximity). Kept 5/32 own memories"`
- What was retrieved: Teamwork goals, house number combinations, IGTV, Chicago theater, estate sale — **none about feed purchases**
- What was needed: Memories about animal feed purchases (answer sessions `answer_92147866_1` and `answer_92147866_2`)
- **Root cause:** 32 memories in the 2-month window, top-5 by time proximity — completely missed the feed purchase memories

### What to change

In `memory-processor-v2.ts`, modify the time-window retrieval path:

**Current flow:**
```
time_frame detected
  → fetch all memories in window (returns N memories)
  → rank by (relevance_score * weight + time_proximity * weight)
  → return top-K
```

**New flow:**
```
time_frame detected
  → fetch all memories in window (returns N memories)  [unchanged]
  → run semantic similarity on those N memories using the primary_query embedding
  → rank by semantic similarity score only (time proximity no longer in ranking)
  → return top-K by semantic score
```

The time window already acts as the time filter — once you're inside the window, recency shouldn't matter anymore. The question "what feed did I buy in the past 2 months?" wants the most semantically relevant memory from that window, not the most recent one.

**Implementation options:**
- Option A: After fetching the time-window memories, re-rank them using cosine similarity against the `primary_query` embedding (embedding is already generated at this point — reuse it)
- Option B: Modify the Supabase time-window function to accept an embedding and apply semantic scoring to the time-filtered results instead of time-proximity scoring

Option A is simpler and keeps the change in TypeScript. Option B requires a DB function change but is more efficient. Choose whichever fits better with the existing code structure.

**Important:** Only change this for the time-window path (`time_retrieval_mode = "all"` with `time_frame` set). The normal semantic path should remain unchanged.

---

## How to verify the fix

After making changes, run the eval on these specific question IDs from Batch 0:

```bash
# From Mercury/evaluation/longmemeval/
npx tsx src/runner.ts eval --batch 0 --question-ids 1d4da289,e48988bc,1c0ddc50,75832dbd,bc149d6b
```

Check the debug output for each:
- `1d4da289`: `retrieved_memories` should include a memory from `answer_ultrachat_348449`
- `e48988bc`: `retrieved_memories` should include a memory from `answer_ultrachat_174360` (Patagonia session)
- `1c0ddc50`: `retrieved_memories` should include a memory from `answer_8da8c7e0` (podcast/audiobook preference)
- `75832dbd`: `retrieved_memories` should include a memory from `answer_d87a6ef8` (AI healthcare research)
- `bc149d6b`: `retrieved_memories` should include a memory from `answer_92147866_1` or `answer_92147866_2` (feed purchases)

Save results to `/Users/openclaw/Desktop/memory-eval-results/` following the existing run structure.

---

## Do Not Change

- The query output schema (`QueryResult` type) — do not add or remove fields
- The normal semantic retrieval path (non-time-window)
- The extraction pipeline
- The eval runner or scoring logic
- `dashboard.html` or any files in `memory-eval-results/`
