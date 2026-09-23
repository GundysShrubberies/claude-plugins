---
description: List and inspect the user's scheduled cloud agents (claude.ai routines), including their models, cron schedules, and enabled state.
---

List the user's **scheduled cloud agents** (claude.ai "routines" — isolated cloud
sessions that fire on a cron schedule) and tell them exactly where to manage each one.
Read-only: this command never creates, edits, runs, or deletes a routine.

## Steps

1. **Load the routine tool.** `RemoteTrigger` is a deferred tool — fetch its schema
   first: `ToolSearch` with query `select:RemoteTrigger`. (Skip if it's already loaded.)

2. **Fetch the list.** Call `RemoteTrigger` with `{action: "list"}`. The response is the
   raw routines JSON. Do **not** use `curl` — the tool handles auth in-process.

3. **Render a scannable table**, one row per routine, sorted by next run (soonest first).
   For each, pull from the JSON:
   - **Name** — `name`
   - **Schedule** — translate `cron_expression` to plain English (`17 * * * *` →
     "hourly at :17", `23 13 * * *` → "daily 13:23 UTC", `0 9 * * 1-5` → "weekdays 09:00
     UTC"). Show `run_once_at` instead for one-shots.
   - **State** — `enabled` (✅ on / ⏸ disabled). Note `ended_reason: "run_once_fired"` as
     "already ran".
   - **Next run** — `next_run_at` (UTC). Omit for fired one-shots.
   - **Model** — `job_config.ccr.session_context.model`
   - **Repos** — the `sources[].git_repository.url` basenames (or "—" if none)
   - **ID** — `id` (the `trig_…` value)

4. **Tell them where to find each one.** Under the table:
   - Per agent: its page is `https://claude.ai/code/routines/{id}` (clickable management +
     run history + last output).
   - The index of all routines: `https://claude.ai/code/routines`.
   - Note that routines **can only be deleted from that web UI**, not from the CLI — but
     they *can* be paused/edited/run-now here via `/schedule` or the `RemoteTrigger` tool
     (`update` / `run`).

5. **Handle the empty / error cases:**
   - **No routines** → say so plainly and point them at `/schedule` to create one.
   - **Auth / API error** → report it verbatim; don't retry silently.

## Notes

- These are **cloud routines** (CCR) — they run on Anthropic's infrastructure on a
  schedule even when this machine is off. This is distinct from background `Agent`/`Task`
  sub-agents spawned *within* a session (those live only in their session). This command
  lists the scheduled cloud kind.
- Keep it terse — a table plus the two URLs. Don't dump the raw JSON or the routines'
  full prompt text unless asked.
- To act on what's listed (pause, change cadence/model, run now, create), hand off to
  `/schedule`.
