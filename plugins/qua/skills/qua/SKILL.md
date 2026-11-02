---
name: qua
description: Run UI checks on the app under test through Qua MCP — perform one prose browser step, run a saved case or suite, or save steps as a case. Use when the user says "check in the browser", "click/fill/verify on the site", "run suite X", or "save this as a case". Not for unit tests or for writing Playwright code.
---

# qua

qua executes browser steps written in plain prose. You write the steps and
call Qua MCP tools; qua finds the element and acts. You never touch the page.

## Project context

Configured browsers, stages and integrations come from the Qua MCP server
instructions (a startup snapshot). Call `qua_project_get` for live state.
Change project data only with `qua_project_set`; never edit `.qua/project.yaml`.

- Omit `browser` to use the default browser.
- Pass `browser` only when the user explicitly asks for one, and only if the
  instructions list it as configured. Otherwise say it is not configured for
  this agent and suggest `qua init`. Do not improvise with another tool.
- Omit `agent`; the server uses the agent it runs under. Your agent is the
  `agent:` line of `qua_project_get`.
- If the instructions say Qua is not configured, tell the user to run `/qua-init`.

## Tools

- `do(action, session, browser?, stage?, target?)` — perform one prose step.
- `run(suite, session, case?, browser?, stage?)` — run a saved case, or every case in a suite.
- `case_write(suite, name, steps)` — save `.qua/{suite}/{name}.md`.
- `qua_session_list()` — list existing sessions.
- `qua_session_close(session)` — only when the user asks to clean up.
- `qua_project_get()` — read project data and whether it is configured.
- `qua_project_set(app_url?, stages?, qua_env?, integrations?, other?)` — merge a patch into project data.
- `init`, `login` — only when the user explicitly asks; both change setup.

## Sessions

- `do`: pick one name per task (for example `signup-flow`) and reuse it for
  every step. The first `do` opens the browser; later calls continue on the
  same page.
- `run`: always a fresh name, for example `smoke-login-20261102-1405`. Names
  already listed by `qua_session_list` are rejected.
- Never close and reopen a session to recover from an error.

## Writing steps

- One browser action per step. Split "fill email and click Next" into two.
- Name what is visible: "Click the Sign in button", not a selector.
- Assertions are steps too: "Verify the heading says Welcome".

## Saving cases

Call `case_write` when the user asks to save, or after they confirm a sequence
you executed with `do`. One step per array item. The same suite and name
overwrite the file.

## Reading results

Each step returns `status`, `reason` and `prose`.

- `passed` — done; continue.
- `review` — qua was not sure and did **not** perform the step. The result
  lists `candidates` (best first: `map_id`, role, name, region, score) and
  likely `actions`. Then:
  1. If exactly one candidate clearly matches what the user meant, call `do`
     again with the same `action`, same `session` and `target: "<map_id>"`.
  2. Otherwise rephrase the step to name the element more precisely and
     retry once, or show the candidates to the user and ask which one.
  Never pick a candidate just because it has the highest score.
- In a `run`, a `review` step stops that case; later cases still run. Report
  the stopped case with its candidates.
- `failed` — report the step and reason to the user. Do not change the app or
  silently edit the case.

## Errors (the tool call itself fails)

- Cursor only (`agent: cursor`): bridge unavailable, wrong host or version →
  ask the user to run `qua patch` and reload the Agents Window, then retry.
  Session lost its tab twice → ask the user to reload the Agents Window, then
  reuse the same session name.
- Playwright endpoint missing or unreachable → ask the user to start the
  browser server or set the endpoint with `qua init`.
- Not logged in or token rejected → ask the user to run `qua login`.
- Qua MCP is not connected → stop and ask the user to fix MCP
  (Claude Code: check `/mcp`; Cursor: check Settings, MCP). Do not fall back to the `qua` CLI.

## Never

- Cursor only (`agent: cursor`): never call these Cursor browser tools: `browser_navigate`, `browser_tabs`, `browser_click`, `browser_type`, `browser_fill`, `browser_snapshot`, `browser_take_screenshot`.
- Never inspect the DOM, pass selectors or page snapshots to qua.
- Never edit `.qua/browser_sessions.json` or other files under `.qua/` except
  through `case_write` and the files a `/qua` command tells you to write
  (`.qua/conventions/*.md`, `.qua/<suite>/suite.yaml`).
- Never ask for or store secrets; `qua login` is run by the user.
- Never run the `qua` CLI during a normal turn; it is for humans.

## Stages

Pass `stage: "<key>"` only when the user names a stage; keys are in the
instructions. It applies when the session is new; in Cursor the tab opens there; Playwright navigates the borrowed page there only when `stage` is passed.
To switch stages, start a new session name.
