---
description: Write a cross-machine handoff for the current session's work to the shared handoff store, so an agent on any other machine can pick it up.
argument-hint: "[slug] [--title T]"
---

Summarise where this session's work stands and write it to the shared handoff
store (`ConnorGunderson/josh-tyler-handoffs` by default — override with the
`KIT_HANDOFF_REPO` env var) with `kit-handoff write`. The reader will be a
fresh agent on a different machine with none of this session's context; the
bar is that they can continue in five minutes without asking anything.

## Locate the scripts first

This plugin's commands can't rely on `${CLAUDE_PLUGIN_ROOT}` — Claude Code
only substitutes that placeholder inside hook/MCP/LSP config and skill/agent
content, not inside a command's own Markdown body. Instead, read the root
this plugin's SessionStart hook recorded at session start:

```sh
cat ~/.claude/gundy-kit/plugin-root
```

That prints `<root>`; the scripts you need are `<root>/scripts/kit-handoff`
and `<root>/scripts/claude-handoff`. If the file is missing or empty (the
plugin was enabled mid-session, before any hook ran), run `/clear` once to
let the hook fire, or find the scripts directly:
`find ~/.claude -maxdepth 6 -name kit-handoff -type f 2>/dev/null`.

## Arguments

`$ARGUMENTS` forms:
- *(empty)* — derive the slug from the work: `<repo>--<topic>` when the work
  is repo-specific (`billing-api--retry-backoff`), else just `<topic>`.
  Kebab-case, lowercase.
- `<slug>` — use this slug. If a handoff with it already exists, this
  **updates it in place** (the store keeps `created`, bumps `updated`).
- `--title T` — override the title (defaults to the body's first `# ` heading).

## Procedure

1. Run `<root>/scripts/kit-handoff template` once if you have not seen the
   section layout for this store (it may 404 if nobody has seeded a template
   file yet — in that case just use the Goal/State/Next/Gotchas/Pointers
   layout below).
2. Write the body from what actually happened in this session:
   - **Goal** — one line: what this piece of work is trying to achieve.
   - **State** — what is *done and verified*. Name files, commits, PR and
     issue numbers, commands that were run and what they returned.
     Unverified work goes in **Next**, not **State**.
   - **Next** — ordered; the first step must be runnable immediately.
   - **Gotchas** — decisions already made *and why*, and approaches that
     failed, so they are not re-litigated.
   - **Pointers** — `owner/repo`, branch, PR/issue links, and
     absolute-enough paths (`<repo>/src/x.ts`, not `./x.ts`).
   - Never include a secret, token, or credential.
3. Write it with the body on stdin, from inside the repo the work belongs to
   when there is one (the script records `repo` and `branch` from `origin`):

   ```sh
   <root>/scripts/kit-handoff write <slug> [--title "..."] <<'EOF'
   # <title>

   ## Goal
   ...
   EOF
   ```

   Outside a repo, pass `--repo owner/repo` and `--branch` explicitly.
4. Report the `HANDOFF:` line the script prints, then **stop**. Do not keep
   working on the task after handing it off — that is what the handoff is for.

Writing is idempotent per slug; picking up (`/pickup`) archives the handoff as
the claim, so two people can never pick up the same one.
