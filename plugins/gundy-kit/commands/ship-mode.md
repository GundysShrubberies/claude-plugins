---
description: Change how /dispatch asks before shipping something — one at a time, all at once, or leave it for the operator.
argument-hint: "[ask|batch|operator]"
---

Show or change `.kit/config.json`'s `ship_gate` — the setting that controls
how `/dispatch` asks a human before it ships (merges) anything. Nothing about
*whether* a human approves ever changes here — only *how* they're asked.
That's a fixed rule of this plugin, not a setting.

## Locate the scripts

Read `~/.claude/gundy-kit/plugin-root` to get `<root>`. If missing/empty, run
`/clear` once, or find it:
`find ~/.claude -maxdepth 6 -name kit-board -type f 2>/dev/null`.

## Arguments

`$ARGUMENTS` is `ask`, `batch`, or `operator` — optional. With none, just
show the current setting and explain the three options; don't change
anything.

## Steps

1. **Find the config.** `.kit/config.json` must exist at the repo root
   (search upward from `$PWD`, same as `kit-board`). If it's missing, say so
   plainly and point at `/start-here` — there's nothing to change yet.

2. **Read the current value** (`jq -r '.ship_gate // "ask"' .kit/config.json`
   — missing means "ask", same default `/dispatch` uses).

3. **No argument given** — report the current setting and explain the three
   in one plain line each, then stop:
   - **ask** — "I ask you about each thing separately, right when it's
     ready."
   - **batch** — "I hold everything that's ready and ask you about all of it
     at once."
   - **operator** — "I don't ask at all — I leave ready work for you to ship
     yourself, whenever you get to it."

4. **An argument was given:**
   - Not one of `ask`/`batch`/`operator` (e.g. someone asks for "auto" or
     "just ship it whenever") — explain plainly that shipping always needs a
     person's explicit yes, in one of these three shapes; there's no setting
     that skips that. Don't change anything.
   - One of the three, and it's already the current value — say so ("already
     set to `<value>`") and stop; no need to rewrite the file.
   - One of the three, different from current — update it:
     ```bash
     tmp=$(mktemp) && jq --arg v "<value>" '.ship_gate = $v' .kit/config.json > "$tmp" && mv "$tmp" .kit/config.json
     ```
     Then confirm in plain English what changed and what it means going
     forward, using the same one-line explanations as step 3.

5. Never touch anything else in `.kit/config.json` — this command's whole
   job is the one field.
