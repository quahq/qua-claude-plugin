---
name: qua-regress
description: Run the project's saved Qua suites, optionally only those tagged for a stage, and summarize the results. Use when the user asks to run regression or the suites, or when /qua-regress is run.
---

# /qua-regress

Run saved suites and summarize. The text after the command is an optional stage key, for example `qa`.

## Rules

- If Qua tools are missing, stop and point the user to /qua-init.
- Call `qua_project_get()` first. If `configured` is false, stop and tell the user to run `/qua-init`.
- Never write to any tracker. Never edit `.qua/project.yaml`.
- Never change cases to make a run pass.

## Steps

1. List the suite directories in `.qua/` that hold case `*.md` files. Skip `conventions` and `runs`.
2. Choose suites:
   - With a stage argument: it must be a key in `stages`, otherwise list the configured keys and stop. Keep only suites whose `.qua/<suite>/suite.yaml` lists that stage under `stages`. Suites with no `suite.yaml` are excluded. If no suite matches, say so and run nothing.
   - Without an argument: keep every suite, tagged or not. Do not pass `stage`.
3. For each chosen suite call `run(suite, session, stage?)`. `session` is a fresh name such as `regress-<suite>-<YYYYMMDD-HHMM>`. Pass `stage` only when the user gave one. Run suites one after another.
4. Summarize:
   - A table with one row per case: suite, case, status (`passed`, `failed` or `review`).
   - For each `failed` case: the step and the reason.
   - For each `review` case: the step and its `candidates` (`map_id`, role, name). Do not pick a candidate yourself.
   - The log paths under `.qua/runs/` when the results give them.
   - Totals: passed, failed, review.
