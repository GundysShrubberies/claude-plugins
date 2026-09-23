---
description: Describe what you want in your own words; Claude turns it into a filed, ready-to-build request.
argument-hint: "[what you want]"
---

The front door for the non-coder collaborator. They describe what they want
in their own words — rough, informal, no technical detail required. Your job
is to turn that into a filed issue they've explicitly agreed to, sitting in
Approved on the board, ready for `/dispatch` to pick up. Never show diffs,
file paths, code, or jargon here unless they specifically ask to see them.

## Locate the scripts

Read `~/.claude/gundy-kit/plugin-root` to get `<root>`; you'll use
`<root>/scripts/kit-board`. If missing/empty, run `/clear` once, or find it:
`find ~/.claude -maxdepth 6 -name kit-board -type f 2>/dev/null`.

## Arguments

`$ARGUMENTS` is what they want, in their own words. If empty, ask them:
"What would you like built?" — one plain question, no options needed.

## Steps

1. **Find the config.** `.kit/config.json` must exist at the repo root. If
   it's missing, tell them plainly this repo isn't set up yet and point them
   at `/start-here` — don't try to guess a project.

2. **Ask at most 2-3 clarifying questions, in plain English.** Only ask what
   you genuinely need to describe the work back to them accurately — skip
   anything you can reasonably infer. Every question needs a recommended
   answer, per your standing instructions:
   - State the recommended option first, label it "(Recommended)", and give
     a one-line plain-English reason.
   - Example shape: "Should this show up on the main page, or somewhere new?
     (Recommended: the main page — it's what people will already be
     looking at.) You can tell me otherwise."
   - Never ask about implementation details (frameworks, data models, file
     structure) — that's your job, not theirs.

3. **Restate the request in plain language and confirm.** Say: "Here's what
   I'll build: <2-4 plain sentences, no jargon, describing the outcome from
   their point of view — what they'll see or be able to do>." Then ask:
   "Should I go ahead and file this?"
   - If they want changes, revise and restate — don't file until they say
     yes.
   - If they say something that sounds like a bug report, still restate it
     as an outcome ("it should do X instead of Y") rather than diagnosing
     out loud.

4. **On yes, file it via the raise-issue flow:**
   - Draft the issue body using the same Goal / Acceptance criteria / Out of
     scope structure as `/raise-issue`, written from what they confirmed in
     step 3 — plain language, no code or paths unless truly necessary.
   - Check for an obvious duplicate first (`gh issue list --repo <repo>
     --state open --limit 100 --json number,title,url`); if you find one,
     tell them in plain terms it looks like this is already in progress and
     don't file a second one.
   - File it: `gh issue create --repo "$REPO_FULL" --title "..." --body-file "..."`
   - Add it to the board: `<root>/scripts/kit-board add "$ISSUE_URL"`
     (defaults to Approved).

5. **Report in plain English and stop:** something like "Done — I've filed
   this and it's ready to build. Run /dispatch (or just say 'go') and I'll
   build it." Do not start building here.
