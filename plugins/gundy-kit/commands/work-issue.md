---
description: Implement a GitHub issue as an open, ready-for-review pull request.
argument-hint: "<issue-number>"
---

Pull a GitHub issue's context and implement it as a pull request, opened and
ready for review. This command **stops once the PR is open** — it never
merges anything, never touches CI (there isn't any here), and never manages
tiers, registries, or runners. Reviewing the PR and deciding whether it ships
are separate steps (`kit-reviewer`, then `/dispatch`'s ship gate).

> **WORKER CONTRACT** — non-interactive execution rules, since this runs
> unattended inside `kit-worker` as often as it runs interactively:
> never pause for a human response — if something is genuinely ambiguous or
> blocking, stop and report it rather than guessing; match the issue by its
> number in the current repo (never assume a bare number means the same
> issue in a different repo).

## Locate the scripts

Read `~/.claude/gundy-kit/plugin-root` to get `<root>`; you'll use
`<root>/scripts/kit-worktree` and `<root>/scripts/kit-board`. If missing/empty,
run `/clear` once, or find it: `find ~/.claude -maxdepth 6 -name kit-board -type f 2>/dev/null`.
(Inside `kit-worker`, `${CLAUDE_PLUGIN_ROOT}` already resolves — use that
instead of the plugin-root file.)

## Arguments

`$ARGUMENTS` is the issue number (e.g. `42` or `#42`). If empty, ask which
issue (interactive use only — `kit-worker` always passes one).

## Steps

1. **Fetch the issue.**
   ```bash
   gh api --method GET repos/<owner>/<repo>/issues/<n>
   gh api --method GET "repos/<owner>/<repo>/issues/<n>/comments?per_page=100"
   ```
   Read the title, body (Goal / Acceptance criteria / Out of scope, if
   `/vibe` or `/raise-issue` filed it), and any comments — later comments can
   settle an open question the body left unanswered.

2. **Check it's not already spoken for.** If the issue already carries the
   `kit-working` label and this run is interactive (not `kit-worker`, which
   is always the one that added it), confirm before continuing — someone
   else may be mid-build. `kit-worker` never needs to check this: `/dispatch`
   already labeled it before spawning the worker.

3. **Branch — an isolated worktree, before touching any files.**
   ```bash
   WT=$(<root>/scripts/kit-worktree "kit/<n>-<short-slug>") && cd "$WT"
   ```
   Prefix every later command with `cd "$WT" &&` (or just stay there). This
   is idempotent — re-running resumes the same worktree if one already
   exists for this branch. Clean up once merged: `git worktree remove "$WT"`.

4. **Plan against the acceptance criteria.** From inside `$WT`, map the
   issue's acceptance criteria to what needs to change. If something is
   genuinely contradictory or missing information that can't reasonably be
   inferred, don't guess at intent that matters — stop and report it as
   blocked (see step 9's failure shape) rather than building the wrong
   thing. A judgment call that's reasonably inferable (naming, exact wording,
   where a thing lives when the issue didn't say) — just make it and note
   the assumption in the PR body later.

5. **Mark it working.**
   ```bash
   gh label create kit-working --color 1d76db --description "Claude is actively building this" --force --repo <owner>/<repo> >/dev/null 2>&1 || true
   gh api --method POST repos/<owner>/<repo>/issues/<n>/labels -f "labels[]=kit-working" >/dev/null 2>&1 || true
   ```
   Move the board card to **In Progress** — best-effort, never blocking:
   ```bash
   <root>/scripts/kit-board move <n> "In Progress" || true
   ```
   (`kit-worker`/`dispatch` will usually have already done both of these —
   that's fine, both calls are idempotent.)

6. **Implement against the acceptance criteria**, matching the surrounding
   code's own style rather than imposing a new one. Keep the change scoped
   to what the issue asks for — note anything adjacent-but-out-of-scope in
   the PR body rather than fixing it inline.

7. **Run QA before drafting the PR.** Follow `/qa`'s procedure exactly (read
   `<root>/commands/qa.md` if the `/qa` command itself isn't directly
   callable from here) against the worktree. **Do not open a PR on a FAIL** —
   fix it and re-run, unless the failure is pre-existing on the base branch
   (confirm by checking `git stash` + re-running QA on the unmodified base,
   or noting it plainly in the PR body as pre-existing and out of scope).

8. **Self-review before committing.** Read your own diff
   (`git diff --stat` then the full diff) against the acceptance criteria
   once more: anything unfinished, any leftover debug output, any secret or
   credential-shaped string, any file deleted that shouldn't be. Fix what you
   find; note anything you deliberately left for the reviewer to weigh in on.

9. **Commit, push, open the PR.**
   ```bash
   git add -A && git commit -m "<one-line summary>"
   git push -u origin "kit/<n>-<short-slug>"
   ```
   Write the PR body to a file first (never `--body` inline — long bodies
   get mangled), starting with a plain-English section and ending with the
   technical one:
   ```markdown
   Closes #<n>

   ## What this changes

   <2-4 plain sentences — what will look/work differently, from the point of
   view of whoever asked for this. No jargon, no file paths, no code.>

   ## Notes

   <Technical notes for the reviewer: key files touched, anything you
   assumed or deliberately left out of scope, and the QA result from step 7.>
   ```
   ```bash
   gh pr create --repo <owner>/<repo> --title "<short imperative title>" --body-file "<path>"
   ```

10. **Stop.** Do not merge, do not wait on any check, do not touch tiers or
    labels beyond `kit-working` (leave that in place — `dispatch`/the
    reviewer manage what happens to it next). If this ran inside
    `kit-worker`, return the fixed-format report from `agents/kit-worker.md`
    now; if run interactively, report the PR URL and a one-line plain-English
    summary of what it does, then stop.

## If it can't be built

If step 4 finds a genuine blocker (contradictory AC, a dependency that
doesn't exist, something only a human can decide), do not open a PR. Report
plainly what's blocking it and what a human needs to decide or provide — this
is what `kit-worker`'s `STATUS: BLOCKED` maps to, which sends the issue to
"Needs help" on the board instead of silently failing.
