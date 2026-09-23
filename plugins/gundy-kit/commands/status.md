---
description: Plain-English summary of what's being built, what's stuck, and what's done recently.
argument-hint: ""
---

Summarize the board's state in plain English — no jargon, no raw command
output. Built for the non-coder collaborator to check in on progress at any
time without needing to understand GitHub Projects.

## Locate the scripts

Read `~/.claude/gundy-kit/plugin-root` to get `<root>`; you'll use
`<root>/scripts/kit-board`. If missing/empty, run `/clear` once, or find it:
`find ~/.claude -maxdepth 6 -name kit-board -type f 2>/dev/null`.

## Steps

1. **Find the config.** If `.kit/config.json` is missing, say so plainly and
   point at `/start-here` — there's no board to summarize yet.

2. **Pull each status column** (`number<TAB>title<TAB>url` per line):
   ```bash
   <root>/scripts/kit-board list "In Progress"
   <root>/scripts/kit-board list "Needs help"
   <root>/scripts/kit-board list Approved
   <root>/scripts/kit-board list Done
   ```
   For "Needs help" items, also check for a comment explaining why (`gh issue
   view <url> --json comments -q '.comments[-1].body'` or similar) so you can
   say *why* it's stuck, not just that it is.

3. **Report in this shape, plain English, no table dumps:**
   ```
   Being built right now: <title> (and any others in "In Progress")
   — or "Nothing being built right now."

   Waiting for help: <title> — <one-line plain reason, if you found one>
   — or "Nothing's stuck."

   Queued up next: <count> thing(s) approved and waiting to start
   — or "Nothing queued."

   Done recently: <up to 3 titles from Done>
   — or "Nothing finished recently."
   ```
   Use titles, not issue numbers, as the primary reference — mention the
   number/URL only if they ask for it or want to open something.

4. If everything is empty across all four, just say: "Nothing on the board
   right now — say /vibe to start something new."
