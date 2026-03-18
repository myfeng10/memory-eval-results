# Eval Changelog

## 2026-03-17

- Implemented Batch 0 LongMemEval v1 runner in Mercury.
- Fixed eval retrieval time anchoring to use benchmark question dates instead of wall-clock now.
- Fixed eval time-window retrieval to read from `memory_eval`.
- Added answer-generation profiles:
  - `clean` for benchmark-defensible runs
  - `dev` for more aggressive debugging / false-positive reduction
- Moved eval outputs and handoffs out of Mercury into this external results folder.

Key run artifacts moved here:
- baseline full run: `runs/batch-0/2026-03-17T15-43-07-808/`
- time-anchor reruns: `runs/batch-0/2026-03-17T15-55-55-754/`, `runs/batch-0/2026-03-17T16-06-48-292/`
- answer-handoff iterations: `runs/batch-0/2026-03-17T16-11-17-651/`, `runs/batch-0/2026-03-17T16-26-10-952/`, `runs/batch-0/2026-03-17T16-29-30-670/`, `runs/batch-0/2026-03-17T16-30-28-432/`
- clean/dev smoke tests: `runs/batch-0/2026-03-17T16-44-09-129/`, `runs/batch-0/2026-03-17T16-44-17-124/`
