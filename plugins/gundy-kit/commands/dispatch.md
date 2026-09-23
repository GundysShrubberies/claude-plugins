---
description: Build everything that's Approved on the board, review it independently, then ask you in plain English before shipping anything.
argument-hint: ""
---

The orchestrator. Pulls everything sitting in **Approved** on the board,
builds each one to a pull request, has an independent reviewer check it, and
then — this is the one rule that governs everything else in this command —
**asks a human, in plain English, before anything merges.** There is no
setting, flag, or failure mode that skips that ask. `ship_gate` (see the
SHIP GATE step and `/ship-mode`) only changes *how* you're asked, never
*whether*.

## Locate the scripts

Read `~/.claude/gundy-kit/plugin-root` to get `<root>`; you'll use
`<root>/scripts/kit-board`, `<root>/scripts/kit-worktree`, and
`<root>/scripts/kit-merge`. If missing/empty, run `/clear` once, or find it:
`find ~/.claude -maxdepth 6 -name kit-board -type f 2>/dev/null`.

## Steps

### 0. Load and validate the config

`.kit/config.json` must exist at the repo root. If it's missing, say so
plainly and point at `/start-here` — don't guess at a repo or project.

Read `repo`, `project_owner`, and `project_number`. **None of these have a
default.** If any is missing, stop with a plain-English error naming exactly
which field is missing and that `.kit/config.json` needs it — never fall
back to guessing a repo or board. (`kit-board` and `kit-merge` already
enforce this on every call; this is the same rule, checked once up front so
you can explain it in plain English instead of relaying a script's error.)

Read `ship_gate` — `jq -r '.ship_gate // "ask"' .kit/config.json`. Missing
means `ask`; that's the only implicit default anywhere in this command.

### 1. Pull the queue

```bash
<root>/scripts/kit-board list Approved
```

Each line is `number<TAB>title<TAB>url`. If empty: "Nothing's waiting to be
built right now — say /vibe to start something new." and stop.

### 2. Build, at most 2 at once

For each Approved item, up to **2 concurrently** (never more — two workers in
the same repo already contend for CI-equivalent local resources; a third
buys nothing and risks worktree collisions):

1. Move it: `<root>/scripts/kit-board move <n> "In Progress"` and label it:
   ```bash
   gh label create kit-working --color 1d76db --description "Claude is actively building this" --force --repo "$REPO" >/dev/null 2>&1 || true
   gh api --method POST "repos/$REPO/issues/<n>/labels" -f "labels[]=kit-working" >/dev/null 2>&1 || true
   ```
2. Spawn a `kit-worker` agent for it (via the `Agent` tool, `subagent_type:
   "kit-worker"`), telling it the issue number, the repo, and the branch name
   to use: `kit/<n>-<short-slug>` (worktree lands at
   `~/.claude/gundy-kit/worktrees/<repo>/kit/<n>-<short-slug>` via
   `kit-worktree` — the worker runs that itself as part of `/work-issue`
   step 3; you don't need to run it yourself). Launch the batch's agents in
   one message so up to 2 run concurrently; wait for both before starting
   the next pair.

When a worker returns:
- **`STATUS: DONE`** — go to step 3 (review) with its `PR`, `BRANCH`,
  `HEAD_SHA`, and `WHAT_CHANGED_PLAIN`.
- **`STATUS: BLOCKED` or `STATUS: FAILED`** — go straight to step 6
  (needs-help handling) with its `NOTES` as the plain-English reason. Do not
  review or ship a PR that never opened.

### 3. Review, independently

For each `STATUS: DONE` worker report, spawn a `kit-reviewer` agent (its own
fresh context — it must not see the worker's reasoning, only the PR and the
issue) with the PR URL and the issue it closes.

- **`VERDICT: BLOCK`** — go to step 6 with the reviewer's `REASONS` as the
  plain-English reason. Leave the PR open; do not close or edit it.
- **`VERDICT: APPROVE`** — post the approval as a PR comment, exactly this
  shape (the `<!-- kit-review: ... -->` line is what `kit-merge` checks for,
  so it must be exact and on its own):
  ```bash
  gh api --method POST "repos/$REPO/issues/<pr>/comments" -f body="<!-- kit-review: APPROVE sha=<head_sha> -->
  $REVIEWER_SUMMARY"
  ```
  Then this item is **ready to ship** — carry its PR URL, issue number, and
  `WHAT_CHANGED_PLAIN`/reviewer `SUMMARY` into the SHIP GATE below.

### 4. The SHIP GATE

Everything reaching this step has an independent APPROVE on its current
head. Nothing here ever merges without a human's explicit yes — `ship_gate`
only changes the shape of the ask.

- **`ship_gate: "ask"` (default)** — one at a time, right as each becomes
  ready. Use `AskUserQuestion`:
  > Ready: **`<one plain sentence — what changed for you, from
  > WHAT_CHANGED_PLAIN>`**. Want me to put it live?
  - **"Yes, ship it (Recommended)"** — it's reviewed and passed QA, this is
    the expected path.
  - **"Show me first"** — explain in one or two plain sentences what it
    does and how to try it locally (e.g. "check out the branch `kit/<n>-…`
    and open it," in terms that fit the project), then **re-ask the same
    question** — don't move on until they say yes or not yet.
  - **"Not yet"** — leave the PR open exactly as it is; move the board card
    back to **In Progress** with a comment noting it's built and reviewed
    but not shipped yet, waiting on you. Do not run `kit-merge`.
  - Only on an explicit **yes** here: `<root>/scripts/kit-merge <pr>`.
    Claude Code then shows one last "Allow?" prompt for that command, because the
    plugin's guard sends every `kit-merge` to a real permission prompt. Tell the
    user first, in plain words: "One last confirm will pop up. Say yes to put it live."

- **`ship_gate: "batch"`** — collect every ready item first, then ask
  **once**, `AskUserQuestion` with `multiSelect: true`, one option per item
  (`"<title> — <one-sentence WHAT_CHANGED_PLAIN>"`), plus recognize that no
  selections is a valid answer (ships nothing). Run `kit-merge <pr>` only
  for the items they picked; the rest stay open exactly as in the "Not yet"
  case above.

- **`ship_gate: "operator"`** — never ask in this session. For each ready
  item:
  ```bash
  gh api --method POST "repos/$REPO/issues/<n>/comments" -f body="Waiting for Connor to approve — it's built and reviewed, ready when you are."
  ```
  Leave the card in **In Progress** and the PR open. The operator ships it
  themselves by running `/dispatch` (or asking a session running as them)
  and answering the same ask/batch question there.

**First ship question in a session only** — right after you ask it (any
mode), print this once, verbatim, so it's never a mystery how to change it:

> You can change this any time: say "ask me one at a time", "ask me once
> for everything", or "let Connor approve", or run /ship-mode. It's the
> `ship_gate` setting in .kit/config.json.

### 5. What "yes" actually does

`<root>/scripts/kit-merge <pr>` re-checks every gate itself (allowlist, open
+ mergeable, `kit/` branch, a fresh APPROVE on the current head, no
`kit-needs-help` label) before it merges — it never trusts this command's own
bookkeeping blindly. On success it squash-merges, deletes the branch, and
moves the issue to **Done**. If `kit-merge` refuses (a gate failed — e.g. the
PR moved to a new commit since it was reviewed), report that plainly and stop
for that item rather than retrying blind; don't run `kit-merge` again without
understanding why it refused.

### 6. Blocked, failed, or reviewer-BLOCKed — needs a human

For anything that didn't make it to a ship decision:
```bash
gh label create kit-needs-help --color b60205 --description "stuck — needs a human to unblock it" --force --repo "$REPO" >/dev/null 2>&1 || true
gh api --method POST "repos/$REPO/issues/<n>/labels" -f "labels[]=kit-needs-help" >/dev/null 2>&1 || true
gh api --method DELETE "repos/$REPO/issues/<n>/labels/kit-working" >/dev/null 2>&1 || true
<root>/scripts/kit-board move <n> "Needs help"
gh api --method POST "repos/$REPO/issues/<n>/comments" -f body="<plain-English explanation of what went wrong and what happens next — e.g. 'I got stuck because ... — once that's sorted, say /dispatch again and I'll pick it back up.'>"
```

### 7. Report back

End with a plain-English summary, no jargon, no raw command output:
```
Built: <n> thing(s) — <titles>
Shipped: <n> thing(s) — <titles>, now live
Waiting on you: <n> thing(s) — <titles, and what you're waiting on: a ship
  answer this run couldn't get to, or "show me first" left unresolved>
Needs help: <n> thing(s) — <titles + one-line plain reason each>
```
Omit a line entirely if its count is zero rather than printing "0 things."

## Changing how shipping works

`ship_gate` in `.kit/config.json` controls how you're asked before anything
ships — never whether. Three values, changeable any time with `/ship-mode`:
**ask** (one at a time, right when ready — the default), **batch** (one
question covering everything that's ready at once), **operator** (don't ask
in this session; leave it for Connor to ship on his own machine). Anything
else someone asks for (e.g. "just ship automatically") isn't a real option —
`/ship-mode` explains why and leaves the setting unchanged.
