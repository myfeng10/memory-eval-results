# Failure Mode Analysis — Batch 0, Run 2026-03-17T15-40-25-520

**Generated:** 2026-03-18  
**Overall accuracy:** 30.43% (7/23 completed)  
**Run config:** gemini-2.5-flash, prompt v1 (factual recall style), dev_memory_eval_hybrid_retrieval_v2

---

## Score Summary

| Question Type          | Score   | Notes |
|------------------------|---------|-------|
| single-session-user    | 4/4 ✅  | Perfect — straightforward fact recall |
| abstention             | 3/3 ✅  | Perfect — correctly said "I don't know" |
| temporal-reasoning     | 2/4     | Mixed — succeeded on simple date math, failed on multi-date reasoning |
| knowledge-update       | 1/4     | Near-misses + stale value retrieval |
| single-session-preference | 0/4  | Zero — prompt fundamentally wrong for this type |
| single-session-assistant  | 0/4  | Zero — reader abstains on assistant-said recall |
| multi-session          | 0/3    | Zero — retrieval gaps + reader abstention |

---

## Failure Mode Taxonomy

### 🔴 A: READER_ABSTAINED_DESPITE_RETRIEVAL
**What it is:** Memories were successfully retrieved (2–3 memories hydrated), but the model answered "I don't have that information." The issue is 100% in the answer prompt — the model was primed to be a factual recall engine, so for anything requiring inference, preference application, or synthesis it bails out.

**Cases (9 total — the dominant failure mode):**

| ID | Type | Question | Memories Found | Hypothesis |
|----|------|----------|---------------|------------|
| 1c0ddc50 | preference | Commute activity suggestions | 3 | ❌ "I don't have that information." |
| 1d4da289 | assistant | What did you say about 2FA methods? | 3 | ❌ "I don't have that information." |
| 2a1811e2 | temporal | Days between Holi and Sunday mass | 3 | ❌ "I don't have that information." |
| 505af2f5 | preference | Coffee creamer recipe suggestions | 3 | ❌ "I don't have that information." |
| 778164c6 | assistant | Name of Jamaican dish from our chat | 3 | ❌ "I don't have that information." |
| 5a7937c8 | multi-session | Faith activities count in December | 3 | ❌ "I don't have that information." |
| fca70973 | preference | Theme park weekend suggestions | 3 | ❌ "I don't have that information." |
| gpt4_385a5000 | temporal | Tomatoes or marigolds started first? | 3 | ❌ "I don't have that information." |
| gpt4_59c863d7 | multi-session | How many model kits total? | 3 | ❌ "I don't have that information." |

**Fix:** Rewrite the answer prompt to handle different question modes (preference, synthesis, temporal reasoning) rather than defaulting to factual recall. Already drafted — needs testing.

---

### 🔴 B: RETRIEVAL_FAILURE_TIME_WINDOW
**What it is:** Query generation correctly detected a time frame, but the retrieval returned 0 memories. The time-window narrowing logic selected no candidates — either the memories weren't extracted into the eval DB for that time range, or the top-K cap (5) cut off all relevant ones.

**Cases (2 total):**

| ID | Type | Question | Time Frame | Memories Found |
|----|------|----------|-----------|---------------|
| 75832dbd | preference | Publications/conferences I'd like | 7 days | ❌ 0 memories |
| bc149d6b | multi-session | Total feed weight past 2 months | 2 months | ❌ 0 memories |

**This is the issue you identified:** retrieval narrowed to a time window, had e.g. 80 candidate sessions, but top-K=5 semantic filter returned nothing relevant. The time-scoped candidates need better post-filtering or a higher K before final re-ranking.

**Fix:** Investigate whether memories for these time windows were extracted at all (Phase A issue) vs. extracted but not surfaced (Phase B top-K issue). Check if increasing K or disabling the semantic threshold for time-windowed queries helps.

---

### 🟠 C: KNOWLEDGE_UPDATE_FAILURE
**What it is:** There are two conflicting memory values (old vs. updated). The retrieval found memories but returned the stale value. The reader used it without checking recency.

**Cases (1 confirmed):**

| ID | Type | Question | Expected | Hypothesis |
|----|------|----------|----------|------------|
| 852ce960 | knowledge-update | Mortgage pre-approval from Wells Fargo | $400,000 | ❌ $350,000 |

**Fix:** The answer prompt should instruct the model to prefer the most recent memory when values conflict. The retrieval ranking may also need to weight recency more heavily for knowledge-update type queries. (This is hard to detect at query time — may need question-type awareness in the pipeline.)

---

### 🟠 D: WRONG_MEMORY_RETRIEVED
**What it is:** Memories were retrieved but the wrong one was surfaced / ranked higher, leading to an incorrect answer. Not an abstention — the model answered confidently with wrong info.

**Cases (1):**

| ID | Type | Question | Expected | Hypothesis |
|----|------|----------|----------|------------|
| e48988bc | assistant | Ethical supply chain company example | Patagonia | ❌ Reebok |

**Note:** Reebok was likely also in the retrieved memories (possibly a different context). The model picked the wrong one. Could be a ranking issue or the model not being careful about which company was said in the context of "environmentally responsible" specifically.

**Fix:** Retrieval re-ranking or prompt instruction to pick the most contextually relevant memory, not just any retrieved one.

---

### 🟡 E: EVALUATOR_NEAR_MISS (False Negatives)
**What it is:** The hypothesis is semantically correct but the evaluator's string normalization / GPT-4o judgment marked it wrong. These may be actual correct answers that should be re-reviewed.

**Cases (3):**

| ID | Type | Expected | Hypothesis | Note |
|----|------|----------|------------|------|
| 2133c1b5 | knowledge-update | "3 months" | "Three months." | Semantically identical |
| 6222b6eb | assistant | "The 6S algorithm is implemented in the SIAC_GEE tool." | "6S is implemented in the SIAC_GEE tool." | Essentially same |
| d7c942c3 | knowledge-update | "Yes." | "Yes, your mom is using the same grocery list app as you." | More verbose but correct |

**Fix:** These are evaluator issues, not pipeline issues. Could add a second-pass review for cases where hypothesis and expected are semantically close but string-normalized differently. May want to re-run with GPT-4o evaluator (LongMemEval standard) to see if these flip to correct.

---

### ⚫ F: FAILED_RUN
**What it is:** A runtime error during the eval run caused the question to be skipped.

**Cases (1):**

| ID | Type | Question | Error |
|----|------|----------|-------|
| d682f1a2 | multi-session | How many food delivery services used recently? | TypeError: 'int' object is not subscriptable |

**Fix:** Debug the specific data shape for this question — something in the retrieval result has an unexpected type. Not a model/prompt issue.

---

## Summary: What to Fix, In Priority Order

| Priority | Failure Mode | Impact | Fix |
|----------|-------------|--------|-----|
| 🔴 1 | Reader abstains despite retrieval (A) | 9 cases | Rewrite answer prompt — already drafted |
| 🔴 2 | Time-window retrieval returns 0 (B) | 2 cases | Investigate top-K cap + time-window logic |
| 🟠 3 | Knowledge update picks stale value (C) | 1 case | Add recency preference in prompt + retrieval |
| 🟠 4 | Wrong memory selected (D) | 1 case | Improve retrieval ranking or prompt guidance |
| 🟡 5 | Evaluator near-misses (E) | 3 cases | Re-run with standard GPT-4o judge; likely free wins |
| ⚫ 6 | Runtime error (F) | 1 case | Debug d682f1a2 data shape |

**Expected gain from fix #1 alone (new prompt):** up to +9 correct → from 30% → ~70% if all A cases flip. Realistically expect 60-65% given some will still be tricky.

**Expected gain from fix #5 (evaluator):** +3 if near-misses flip → ~83% ceiling for this run's data quality.

---

## Notes for Next Run
- Test new answer prompt on Batch 0 (same questions) to measure delta
- Specifically watch: preference questions (0→?), assistant questions (0→?), temporal reasoning (50%→?)
- Flag any new cases where hypothesis looks right but eval says wrong — may be more evaluator issues
