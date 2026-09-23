---
description: Draft a well-formed GitHub issue from a rough description, file it, and add it to the board's Approved column.
argument-hint: "<rough-description>"
---

Draft a well-formed GitHub issue from a rough description, file it with `gh`,
add it to the shared board in Approved, report the issue URL, and **stop**.
This is a simplified, single-repo version of a fuller raise-issue
flow — no risk tiers, no umbrella/workstream/parked handling, no extra
labels beyond what `kit-setup` already created. This command is for the
technical collaborator working directly in a repo; `/vibe` is the front door
for the non-coder collaborator and calls into this same mechanism after it
has confirmed the request in plain English.

## Locate the scripts

Read `~/.claude/gundy-kit/plugin-root` (a SessionStart hook writes it) to get
`<root>`; the script you need is `<root>/scripts/kit-board`. If the file is
missing or empty, run `/clear` once to let the hook fire, or find the script
directly: `find ~/.claude -maxdepth 6 -name kit-board -type f 2>/dev/null`.

## Arguments

`$ARGUMENTS` is the rough description of what's needed. If empty, ask the
user for it in one line — don't guess at scope.

## Steps

1. **Find the config.** `.kit/config.json` must exist at the repo root (search
   upward from `$PWD`, same as `kit-board` does). If it's missing, stop and
   tell the user to run `kit-setup` first (or `/start-here` if this is their
   first time) — don't invent a repo or project number.

2. **Ground it in the real repo.** Read `CLAUDE.md` (if present) and skim the
   handful of files most relevant to the request, so the issue references
   real paths, not guesses. A few targeted reads — don't over-research.

3. **Check for an existing issue that already covers this**, so you don't
   file a duplicate:
   ```bash
   REPO_FULL=$(jq -r .repo .kit/config.json)   # or wherever load_config found it
   gh issue list --repo "$REPO_FULL" --state open   --limit 100 --json number,title,url
   gh issue list --repo "$REPO_FULL" --state closed --limit 100 --json number,title,url,closedAt
   ```
   - **Clear duplicate** → don't file a second one. Tell the user which issue
     already covers it and its URL, and stop.
   - **Ambiguous overlap** → ask the user whether this is the same request or
     genuinely separate — don't silently merge on a guess.
   - **Nothing related** → proceed.

4. **Draft the issue body**, plain and short — this reader may not be
   technical, so keep jargon out of the body itself:
   ```
   ## Goal
   <one or two sentences: what should exist/work when this is done, and why>

   ## Acceptance criteria
   - [ ] <a specific, checkable outcome>
   - [ ] <another — 2 to 6 total, each independently verifiable>

   ## Out of scope
   <one line, or "Nothing specific" — what this issue deliberately does not cover>
   ```
   Save it to a temp file — pass `--body-file`, not an inline `--body`, to
   avoid shell-quoting problems on multi-line markdown.

5. **File it and add it to the board:**
   ```bash
   ISSUE_URL=$(gh issue create --repo "$REPO_FULL" --title "<short, specific title>" --body-file "<path>")
   <root>/scripts/kit-board add "$ISSUE_URL"      # defaults to Approved
   ```

6. **Report and stop.** Give the user the issue URL and a one-line summary
   of what you filed. Don't hand off to building it — that's a separate,
   deliberate step (`/dispatch`, or saying "go").
