---
description: Turn guided mode on or off, or check whether it's currently on.
argument-hint: "on|off|status"
---

Toggle guided mode: a flag file whose presence tells every session (via the
`kit-guided-context` SessionStart hook) to explain things in plain English,
avoid jargon, and ask before doing anything outward-facing. Meant for the
non-coder collaborator; the technical collaborator will usually leave it off.

## Arguments

`$ARGUMENTS` is `on`, `off`, or `status`. If empty, treat it as `status`.

## Steps

The flag file is `~/.claude/gundy-kit/guided` — its mere existence means "on"
(contents don't matter).

- **`on`**: `mkdir -p ~/.claude/gundy-kit && touch ~/.claude/gundy-kit/guided`.
  Confirm: "Guided mode is on. From now on I'll explain things in plain
  English and check with you before doing anything outward-facing." This
  takes effect starting with the *next* session (or after `/clear`) — say so
  if it matters for what they're about to do next.

- **`off`**: `rm -f ~/.claude/gundy-kit/guided`. Confirm: "Guided mode is
  off."

- **`status`**: report whether the file exists, in one line — "Guided mode
  is currently on/off."

Keep the confirmation to one or two short sentences either way — no need to
explain the mechanism unless asked.
