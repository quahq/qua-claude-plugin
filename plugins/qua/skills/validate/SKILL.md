---
description: Reproduce a ticket in the live app with Qua and report a verdict with evidence in chat. Use when the user asks to validate, verify, retest or reproduce a ticket, or when /qua:validate is run.
---

# /qua:validate

Validate one ticket live. The text after the command is the ticket key or URL, for example `ADSAI-123`. If it is empty, ask for it.

## Rules

- If Qua tools are missing, stop and point the user to /qua:init.
- Call `qua_project_get()` first. If `configured` is false, stop and tell the user to run `/qua:init`.
- The issue tracker is read-only. Never comment, transition or edit the ticket without a separate, explicit yes after the verdict.
- Never edit `.qua/project.yaml` and never handle secrets.

## Steps

1. Read the ticket through the issue tracker tools you have (`integrations.issue_tracker`). If you cannot read it, say so and stop.
2. Derive numbered repro steps from the summary, description and comments: the starting page, the actions, the expected and the actual result. If the steps are ambiguous, ask one question.
3. Choose the stage: the one the ticket names, if it matches a key in `stages`; otherwise omit `stage` to use `app_url`.
4. Reproduce it on a new session name such as `validate-<ticket>-<HHMM>`. Run each action with `do(action, session, agent, stage?)`: one browser action per call, assertions as separate steps. `agent` is your own harness name: `claude` or `cursor`. Pass `stage` only on the first call. Follow the `qua` skill for review results and errors.
5. Report in chat:
   - Verdict: `reproduced` (the bug happens), `not reproduced` (the expected behavior holds), or `blocked` (you could not reach the point of the check: access, data, environment, a failed step you cannot explain).
   - Evidence: the steps you ran, each step's status, the observed result against the expected one, and the session name.
   - For `blocked`: what stopped you.
6. Offer, but do not do, an issue tracker comment. Write to the issue tracker only after a separate explicit yes.
