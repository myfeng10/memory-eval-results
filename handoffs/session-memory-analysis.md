# Agent Handoff: Extraction Audit for Retrieval-Failure Cases

**Date:** 2026-03-18  
**Goal:** For the 6 questions where retrieval failed, determine whether the right memory was ever extracted into the DB. If it was extracted, mark it. If not, find the session in the benchmark that contains the answer and flag it as a missing extraction.

---

## Context: What This Is NOT

This is NOT about the eval run results or the answer prompt. It's a ground truth audit:
> "Does `memory_eval` contain the memory needed to answer this question?"

The eval run data (debug JSONs, hypothesis, self_check) is not needed here.

---

## Scope: 6 Questions Only

Only these questions — pulled from the dashboard reason buckets `retrieval-off-target-or-incomplete` and `time-window-topk-miss`:

From `dashboard.html`, filter current batch rows where `data-reason` is one of:
- `retrieval-off-target-or-incomplete`
- `time-window-topk-miss`

This should yield exactly 6 question IDs. Read them from the dashboard rather than hardcoding, in case the dashboard was updated.

---

## Data Sources

1. **Benchmark JSON:** `/Users/openclaw/Desktop/Mercury/evaluation/longmemeval/data/longmemeval_s_cleaned.json`
   - Contains per question: `question`, `expected_answer`, `answer_session_ids`, `mock_user_id`, `haystack_sessions` (all sessions with conversation turns)

2. **Supabase `memory_eval` table:** Use env vars from `/Users/openclaw/Desktop/Mercury/.env.local` (`NEXT_PUBLIC_SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`)
   - Query: `SELECT * FROM memory_eval WHERE user_id = '<mock_user_id>' AND haystack_session_id = ANY(ARRAY['<answer_session_id_1>', ...])`
   - This returns only memories extracted from the answer session(s) — targeted, not all 126 memories for the user

---

## Per-Question Process

For each of the 6 questions:

### Step 1: Query Supabase for memories from the answer sessions
```sql
SELECT id, haystack_session_id, description, details, keys, category, object
FROM memory_eval
WHERE user_id = '<mock_user_id>'
AND haystack_session_id = ANY(ARRAY['<answer_session_id_1>', '<answer_session_id_2>'])
```

### Step 2: Check if any memory is helpful
Read each returned memory's `description` and `details`. Mark as `helpful: true` if the content is relevant to answering the question (compare against `expected_answer` — does the memory contain the key fact/preference needed?).

### Step 3: If no helpful memory found in DB
Read the answer session conversation turns from `haystack_sessions` in the benchmark JSON (only the `answer_session_ids` sessions, not all 48). Write a 1-sentence summary of what that session contains and why it should have produced a memory.

### Step 4: Determine the diagnosis
- `EXTRACTED_HELPFUL` — at least one memory is helpful → extraction worked, issue is retrieval ranking/logic
- `EXTRACTED_BUT_UNHELPFUL` — memories exist from the session but none are helpful → extraction captured the wrong info from this session  
- `NOT_EXTRACTED` — no memories at all from the answer session → extraction missed this session entirely

---

## Output

### File: `session-analysis.json`
Save to `/Users/openclaw/Desktop/memory-eval-results/session-analysis.json`

```json
{
  "generated_at": "2026-03-18T...",
  "scope": "retrieval-failure cases only",
  "questions": [
    {
      "question_id": "75832dbd",
      "question_type": "single-session-preference",
      "question": "Can you recommend some recent publications or conferences...",
      "expected_answer": "...",
      "answer_session_ids": ["answer_xxxx"],
      "mock_user_id": "...",
      "memories_from_answer_sessions": [
        {
          "memory_id": "...",
          "haystack_session_id": "answer_xxxx",
          "description": "...",
          "details": "...",
          "keys": "...",
          "helpful": true,
          "helpful_note": "Contains user's interest in AI healthcare research — directly relevant"
        }
      ],
      "diagnosis": "EXTRACTED_HELPFUL",
      "diagnosis_note": "Memory exists and is helpful. Retrieval missed it due to time-window top-K cutoff.",
      "answer_session_summary": null
    },
    {
      "question_id": "bc149d6b",
      "question_type": "multi-session",
      "question": "What is the total weight of the new feed I purchased in the past two months?",
      "expected_answer": "70 pounds",
      "answer_session_ids": ["answer_yyyy"],
      "mock_user_id": "...",
      "memories_from_answer_sessions": [],
      "diagnosis": "NOT_EXTRACTED",
      "diagnosis_note": "No memories found in memory_eval from the answer session.",
      "answer_session_summary": "User discussed purchasing 40 lbs of chicken feed and 30 lbs of rabbit feed in separate conversations over two months."
    }
  ]
}
```

### Dashboard update (`dashboard.html`)
For each of the 6 questions, add a small inline panel inside the existing question row (after the reason tag). **Do not restructure the existing table, filters, or layout.**

Display:
```
Extraction audit:
  [EXTRACTED_HELPFUL] Memory found: "User interested in AI healthcare research..." ✅
  → Retrieval issue, not extraction
```
or:
```
Extraction audit:
  [NOT_EXTRACTED] No memory from answer session found in DB
  Answer session: "User purchased 40 lbs chicken feed, 30 lbs rabbit feed..."
  → Extraction gap — this session was missed
```

Use the existing CSS classes: `.tag`, `.small`, `.status-ok`, `.status-miss`, `.panel`. Inline `session-analysis.json` data as a JS object in the HTML, keyed by `question_id`.

---

## Files to Read First

1. `/Users/openclaw/Desktop/memory-eval-results/dashboard.html` — to extract the 6 question IDs and their canonical run info
2. `/Users/openclaw/Desktop/Mercury/.env.local` — Supabase creds
3. `/Users/openclaw/Desktop/Mercury/evaluation/longmemeval/data/longmemeval_s_cleaned.json` — benchmark data (large file, only read entries for the 6 question IDs)
