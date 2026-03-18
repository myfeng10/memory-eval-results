# Current Batch Analysis

Generated from the synthesized current batch view in `/Users/openclaw/Desktop/memory-eval-results`.

Overall: 12/24 (0.5)
Hits: 12
Misses: 12
Failed: 0
Dry-run only: 0

## Miss Buckets
- `retrieval-off-target-or-incomplete`: 7
- `explicit-time-window-miss`: 2
- `answer-ignored-direct-evidence`: 1
- `answer-misused-retrieved-evidence`: 1
- `retrieval-partial-or-off-target`: 1

## Misses
- [`852ce960`](question-pages/batch-0/2026-03-17T15-43-07-808/852ce960.html) (knowledge update, 2026-03-17T15-43-07-808): retrieval-off-target-or-incomplete. Retrieved 3 memories, but only part of the answer-session set was covered (answer_3a6f1e82_1 of answer_3a6f1e82_1, answer_3a6f1e82_2). Retrieval coverage: 1/2 answer sessions; answer-session memories 3/3.
- [`5a7937c8`](question-pages/batch-0/2026-03-17T15-43-07-808/5a7937c8.html) (multi session, 2026-03-17T15-43-07-808): retrieval-off-target-or-incomplete. Retrieved 3 memories, but only part of the answer-session set was covered (answer_4cef8a3c_2 of answer_4cef8a3c_3, answer_4cef8a3c_1, answer_4cef8a3c_2). Retrieval coverage: 1/3 answer sessions; answer-session memories 3/3.
- [`bc149d6b`](question-pages/batch-0/2026-03-17T16-06-48-292/bc149d6b.html) (multi session, 2026-03-17T16-06-48-292): explicit-time-window-miss. An explicit time window was applied (all, frame=2) and returned 5 memories, but none were from the answer session. This often indicates windowing or top-k/ranking loss. Retrieval coverage: 0/2 answer sessions; answer-session memories 0/5.
- [`d682f1a2`](question-pages/batch-0/2026-03-17T15-55-55-754/d682f1a2.html) (multi session, 2026-03-17T15-55-55-754): retrieval-partial-or-off-target. Retrieved 5 memories, but not the gold session; retrieval status suggests the search broadened or reranked away from the needed evidence. Retrieval coverage: 0/3 answer sessions; answer-session memories 0/5.
- [`gpt4_59c863d7`](question-pages/batch-0/2026-03-17T15-43-07-808/gpt4_59c863d7.html) (multi session, 2026-03-17T15-43-07-808): retrieval-off-target-or-incomplete. Retrieved 3 memories, but only part of the answer-session set was covered (answer_593bdffd_4, answer_593bdffd_1 of answer_593bdffd_4, answer_593bdffd_1, answer_593bdffd_3). Retrieval coverage: 2/4 answer sessions; answer-session memories 3/3.
- [`1d4da289`](question-pages/batch-0/2026-03-17T15-43-07-808/1d4da289.html) (single session assistant, 2026-03-17T15-43-07-808): retrieval-off-target-or-incomplete. Retrieved 3 memories, but none matched the answer session (answer_ultrachat_348449). Retrieval coverage: 0/1 answer sessions; answer-session memories 0/3.
- [`e48988bc`](question-pages/batch-0/2026-03-17T15-43-07-808/e48988bc.html) (single session assistant, 2026-03-17T15-43-07-808): retrieval-off-target-or-incomplete. Retrieved 3 memories, but none matched the answer session (answer_ultrachat_174360). Retrieval coverage: 0/1 answer sessions; answer-session memories 0/3.
- [`1c0ddc50`](question-pages/batch-0/2026-03-17T15-43-07-808/1c0ddc50.html) (single session preference, 2026-03-17T15-43-07-808): retrieval-off-target-or-incomplete. Retrieved 3 memories, but none matched the answer session (answer_8da8c7e0). Retrieval coverage: 0/1 answer sessions; answer-session memories 0/3.
- [`505af2f5`](question-pages/batch-0/2026-03-17T16-30-28-432/505af2f5.html) (single session preference, 2026-03-17T16-30-28-432): answer-ignored-direct-evidence. All answer sessions were retrieved (answer_f3164f2c), but the answer still abstained. Retrieval coverage: 1/1 answer sessions; answer-session memories 2/3.
- [`75832dbd`](question-pages/batch-0/2026-03-17T16-06-48-292/75832dbd.html) (single session preference, 2026-03-17T16-06-48-292): explicit-time-window-miss. An explicit time window was applied (all, frame=7) and returned 5 memories, but none were from the answer session. This often indicates windowing or top-k/ranking loss. Retrieval coverage: 0/1 answer sessions; answer-session memories 0/5.
- [`fca70973`](question-pages/batch-0/2026-03-17T16-30-28-432/fca70973.html) (single session preference, 2026-03-17T16-30-28-432): answer-misused-retrieved-evidence. All answer sessions were retrieved (answer_a1e169b1), but the final answer still missed the expected content. Retrieval coverage: 1/1 answer sessions; answer-session memories 2/3.
- [`2a1811e2`](question-pages/batch-0/2026-03-17T15-43-07-808/2a1811e2.html) (temporal reasoning, 2026-03-17T15-43-07-808): retrieval-off-target-or-incomplete. Retrieved 3 memories, but only part of the answer-session set was covered (answer_1cc3cd0c_1 of answer_1cc3cd0c_1, answer_1cc3cd0c_2). Retrieval coverage: 1/2 answer sessions; answer-session memories 2/3.

## Retrieval Coverage
- `full-answer-session-coverage`: 13
- `no-answer-session-coverage`: 6
- `partial-answer-session-coverage`: 5

## Hits
- [`2133c1b5`](question-pages/batch-0/2026-03-17T16-29-30-670/2133c1b5.html) (knowledge update, 2026-03-17T16-29-30-670): direct-evidence-hit. Retrieved all answer sessions (answer_52382508_1, answer_52382508_2) and produced a matching answer. Retrieval coverage: 2/2 answer sessions; answer-session memories 3/3.
- [`6aeb4375_abs`](question-pages/batch-0/2026-03-17T15-43-07-808/6aeb4375_abs.html) (knowledge update, 2026-03-17T15-43-07-808): correct-abstention. Correctly abstained because the benchmark answer for this case is no-information. Retrieval coverage: 1/2 answer sessions; answer-session memories 1/3.
- [`d7c942c3`](question-pages/batch-0/2026-03-17T16-44-17-124/d7c942c3.html) (knowledge update, 2026-03-17T16-44-17-124): direct-evidence-hit. Retrieved all answer sessions (answer_eecb10d9_1, answer_eecb10d9_2) and produced a matching answer. Retrieval coverage: 2/2 answer sessions; answer-session memories 3/3.
- [`6222b6eb`](question-pages/batch-0/2026-03-17T16-26-10-952/6222b6eb.html) (single session assistant, 2026-03-17T16-26-10-952): direct-evidence-hit. Retrieved all answer sessions (answer_sharegpt_H9PiM5G_0) and produced a matching answer. Retrieval coverage: 1/1 answer sessions; answer-session memories 3/3.
- [`778164c6`](question-pages/batch-0/2026-03-17T16-29-30-670/778164c6.html) (single session assistant, 2026-03-17T16-29-30-670): direct-evidence-hit. Retrieved all answer sessions (answer_ultrachat_399000) and produced a matching answer. Retrieval coverage: 1/1 answer sessions; answer-session memories 3/3.
- [`6ade9755`](question-pages/batch-0/2026-03-17T15-43-07-808/6ade9755.html) (single session user, 2026-03-17T15-43-07-808): direct-evidence-hit. Retrieved all answer sessions (answer_9398da02) and produced a matching answer. Retrieval coverage: 1/1 answer sessions; answer-session memories 3/3.
- [`94f70d80`](question-pages/batch-0/2026-03-17T15-43-07-808/94f70d80.html) (single session user, 2026-03-17T15-43-07-808): direct-evidence-hit. Retrieved all answer sessions (answer_c63c0458) and produced a matching answer. Retrieval coverage: 1/1 answer sessions; answer-session memories 3/3.
- [`bc8a6e93`](question-pages/batch-0/2026-03-17T15-43-07-808/bc8a6e93.html) (single session user, 2026-03-17T15-43-07-808): direct-evidence-hit. Retrieved all answer sessions (answer_e6143162) and produced a matching answer. Retrieval coverage: 1/1 answer sessions; answer-session memories 2/3.
- [`bc8a6e93_abs`](question-pages/batch-0/2026-03-17T15-43-07-808/bc8a6e93_abs.html) (single session user, 2026-03-17T15-43-07-808): correct-abstention. Correctly abstained because the benchmark answer for this case is no-information. Retrieval coverage: 1/1 answer sessions; answer-session memories 3/3.
- [`c8090214_abs`](question-pages/batch-0/2026-03-17T15-43-07-808/c8090214_abs.html) (temporal reasoning, 2026-03-17T15-43-07-808): correct-abstention. Correctly abstained because the benchmark answer for this case is no-information. Retrieval coverage: 2/2 answer sessions; answer-session memories 3/3.
- [`gpt4_385a5000`](question-pages/batch-0/2026-03-17T15-43-07-808/gpt4_385a5000.html) (temporal reasoning, 2026-03-17T15-43-07-808): direct-evidence-hit. Retrieved all answer sessions (answer_7a4a93f1_2, answer_7a4a93f1_1) and produced a matching answer. Retrieval coverage: 2/2 answer sessions; answer-session memories 3/3.
- [`gpt4_fa19884c`](question-pages/batch-0/2026-03-17T15-43-07-808/gpt4_fa19884c.html) (temporal reasoning, 2026-03-17T15-43-07-808): direct-evidence-hit. Retrieved all answer sessions (answer_ff201786_1, answer_ff201786_2) and produced a matching answer. Retrieval coverage: 2/2 answer sessions; answer-session memories 3/3.

## Failed
- none
