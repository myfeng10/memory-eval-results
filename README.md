# Memory Eval Results

Evaluation outputs and notes for Mercury's memory retrieval and answering pipeline live here, outside the Mercury repo.

Benchmark focus:
- LongMemEval-S / Batch 0 work

## Structure

- `runs/`
  - timestamped run directories written automatically by the Mercury eval runner
  - current layout is `runs/batch-0/<timestamp>/`
- `handoffs/`
  - dated agent handoffs and session notes
- `eval-changelog.md`
  - high-level timeline of important evaluation iterations
- `extraction-failures.json`
  - aggregate record of extraction failures across runs

## Source Code

The eval pipeline code stays in Mercury:

`/Users/openclaw/.codex/worktrees/7eb7/Mercury/evaluation/longmemeval/`

## Default Output Path

Mercury writes here by default through `EVAL_OUTPUT_DIR`, falling back to:

`/Users/openclaw/Desktop/memory-eval-results/`

Override example:

```bash
EVAL_OUTPUT_DIR=/some/other/path npx tsx evaluation/longmemeval/src/runner.ts eval --batch 0
```

## Notes

- Keep this directory as results/notes only.
- Do not move evaluation code here.
- For benchmark-clean reporting, prefer the `clean` answer profile.
- For development/debugging, use `--answer-profile dev` when needed.
