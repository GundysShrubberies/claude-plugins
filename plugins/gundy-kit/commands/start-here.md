---
description: First-run walkthrough for a non-coder collaborator — checks access, sets up the repo, turns on guided mode, and runs a tiny first request end to end.
argument-hint: ""
---

A friendly, step-by-step first-run walkthrough for someone who doesn't code.
One thing at a time — do a step, explain what happened in plain English,
then move to the next. Never dump raw command output at them; translate it.

## Locate the scripts

Read `~/.claude/gundy-kit/plugin-root` to get `<root>`; scripts you'll need
live at `<root>/scripts/`. If that file is missing/empty, run `/clear` once
to let the SessionStart hook fire, or find them directly:
`find ~/.claude -maxdepth 6 -name kit-setup -type f 2>/dev/null`.

## Steps

1. **Say hello and set expectations**, briefly: this will check a few things
   are set up, then walk through one small example so they can see how it
   works — a few minutes, one step at a time.

2. **Check GitHub access:** `gh auth status`. Translate the result:
   - Signed in → "Good, you're signed in to GitHub as <name>."
   - Not signed in → explain in one line what that means ("the tool that
     lets me file and build work on GitHub") and offer to run `gh auth
     login` interactively, or point them at asking Connor if that's blocked.

3. **Check the repo is cloned.** Look for `~/josh-tyler-workspace`. If it's
   not there, offer (Recommended, since it's the one they'll be working in):
   "I don't see the project on this machine yet — want me to download a copy
   to ~/josh-tyler-workspace?" On yes: `git clone
   https://github.com/GundysShrubberies/josh-tyler-workspace
   ~/josh-tyler-workspace`. From here on, work inside that directory.

4. **Check `.kit/config.json` exists** in the repo. If missing:
   - If they know the project number, run `<root>/scripts/kit-setup
     --project-number N`.
   - If they don't know it, say plainly: "This project needs one more thing
     from Connor — the number of the board it should use. Ask him, then come
     back and tell me the number." Stop here until they have it; don't
     guess or default to any number.
   - Report what `kit-setup` found in plain English (it ends by running
     `kit-board check`) — if anything's still missing (e.g. the board itself
     doesn't exist yet), say exactly what's missing and who to ask, then
     stop.

5. **Turn on guided mode:** run `/guided on` (or its underlying steps —
   `mkdir -p ~/.claude/gundy-kit && touch ~/.claude/gundy-kit/guided`).
   Explain in one line: "I'll now always explain things in plain English and
   check with you before doing anything outward-facing."

6. **Walk through one tiny first request end to end**, so they see the whole
   loop once:
   - Ask: "Let's try it with something small. What's one little thing you'd
     like changed or added?" If they're unsure, suggest something low-risk
     and visible (Recommended: a text/wording change — it's easy to see
     worked and low-risk if something's off).
   - Run the `/vibe` flow inline: ask your 2-3 clarifying questions (each
     with a recommended answer first), restate as "Here's what I'll build:
     …", get their yes, then file it and add it to the board exactly as
     `/vibe` does.
   - Tell them: "That's filed and ready. Next time, you'd just run /vibe (or
     tell me what you want) to do this again. When you're ready to have me
     actually build it, run /dispatch or just say 'go'."

7. **Close out:** remind them of the two commands they'll use day to day —
   `/vibe` for new ideas, `/status` to check on progress — and that guided
   mode is now on, so every future session will talk to them this way.
