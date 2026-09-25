---
description: List open cross-machine handoffs, or pick one up — which archives it as the claim — and continue its work in this session.
argument-hint: "[slug]"
---

Continue work another agent handed off through the shared store
(`ConnorGunderson/josh-tyler-handoffs` by default — override with
`KIT_HANDOFF_REPO`). **Picking up claims it:** the handoff is archived in the
same command that prints it, before any work starts, so no one else can also
pick it up. Look with `read`; claim with `pickup`.

## Locate the scripts first

This plugin's commands can't rely on `${CLAUDE_PLUGIN_ROOT}` in their own
Markdown body (it's only substituted in hook/MCP/LSP config and skill/agent
content). Read the root this plugin's SessionStart hook recorded:

```sh
cat ~/.claude/gundy-kit/plugin-root
```

That prints `<root>`; the script you need is `<root>/scripts/kit-handoff`. If
the file is missing or empty, run `/clear` once to let the hook fire, or find
the script directly: `find ~/.claude -maxdepth 6 -name kit-handoff -type f 2>/dev/null`.

## Arguments

`$ARGUMENTS` forms:
- *(empty)* — `<root>/scripts/kit-handoff list`. Show the table and, if one
  handoff obviously matches this session's repo or the user's stated intent,
  say which and ask before claiming. Never claim without a slug from the user.
- `<slug>` — claim and continue (procedure below).

## Procedure

1. `<root>/scripts/kit-handoff pickup <slug>`. This prints the handoff and
   moves it to `archive/YYYY/MM/` with `picked_up_by`/`picked_up_at` in its
   frontmatter.
   - Exit 1: no such open handoff — run `list`, it may already be archived.
   - Exit 3: someone else claimed it between listing and now. Do not retry;
     tell the user and stop.
2. Read the whole thing before touching anything. **State** is what the
   writer verified; **Gotchas** holds decisions already made — do not
   re-litigate them without a reason the writer did not have.
3. Go to the repo and branch named in **Pointers**. If you don't already have
   a local clone, clone it; use a separate branch/worktree if you don't want
   to disturb other work in progress there.
4. Work through **Next** in order. Verify each step the same way the handoff
   says the previous ones were verified.
5. If you stop before **Next** is empty — blocked, out of session, escalated
   — hand it back: `/handoff <same-slug>` with the updated State/Next. The
   archived copy stays as history; the new one is the open handoff.

Only `read` (`<root>/scripts/kit-handoff read <slug>`) is safe for "just
looking"; the picked-up copy is gone from the inbox the moment step 1 succeeds.
