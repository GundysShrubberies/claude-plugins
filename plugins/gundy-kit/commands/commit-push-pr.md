---
description: Take the current working-tree changes from commit through to an open pull request, ready for review.
argument-hint: "[pr-title]"
---

Take the current working-tree changes from commit through to an open pull
request, in one flow. Invoking this command is the user's explicit authorization
to commit and push.

## Arguments

`$ARGUMENTS`, if present, is a hint for the PR title / focus. Otherwise infer
everything from the diff.

## Steps

1. **Survey the changes.** Run `git status` and `git diff` (and `git diff
   --staged`) to see everything that will go in. Also run `git log --oneline -5`
   to match the repo's commit message style. If there are zero changes, say so
   and stop.

2. **Check for a linked issue.** Every PR should be backed by a tracked issue on
   the project board. Look for one in two places:
   - Branch name — extract a number from patterns like `issue-3-*`, `feat/3-*`,
     `fix/3-*`, or any branch containing a digit sequence after a `/` or `-`.
   - Recent commit messages — scan for `Closes #N`, `Fixes #N`, or `Refs #N`.

   Extract the issue number as the **first digit run** in the branch name after
   a `/` or `-` separator (e.g. `issue-42-slug` → `42`, `feat/42-title` → `42`).
   Verify it exists:
   ```bash
   gh issue view <N> --repo <current-repo>
   ```
   and note it — you'll add `Closes #N` to the PR body in step 6.

   If **no issue is found**, stop and warn:
   > "No linked issue found. This PR won't appear on the project board. Run
   > `/raise-issue` first (recommended), or type **proceed** to ship without one."

   Wait for the user's response before continuing. If they say proceed, note the
   omission in your final report but continue. Do not silently skip this check.

3. **Branch if needed.** Check the current branch. If you're on the default
   branch (`main`/`master`), create and switch to a new descriptive branch first
   (e.g. `feat/...`, `fix/...`) — never commit directly to the default branch.
   If already on a feature branch, stay on it.

4. **Commit.** Stage the relevant changes (`git add`), then write a concise,
   meaningful commit message describing *why*, not just *what*. End the commit
   message with:

   ```
   Co-Authored-By: Claude <noreply@anthropic.com>
   ```
   (Drop the model name — a generic-but-true trailer beats a specific-but-wrong one;
   the model running at commit time may differ from any hardcoded alias.)

5. **Push.** Abort if still on the default branch before pushing — committing
   directly to `master`/`main` bypasses the PR review gate:
   ```bash
   CURRENT=$(git branch --show-current)
   DEFAULT=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@refs/remotes/origin/@@')
   [ "$CURRENT" = "$DEFAULT" ] && { echo "ERROR: still on default branch $DEFAULT — create a feature branch first"; exit 1; }
   ```
   Then push with upstream tracking: `git push -u origin <branch>`.

6. **Open the PR.** Use `gh pr create`. Write a clear title and a body with a
   short summary and a bullet list of changes. If a linked issue was found in
   step 2, include `Closes #N` in the body. If a PR already exists for the
   branch, skip creation and just report its URL. End the PR body with:

   ```
   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   ```

7. **Report.** Print the branch name, the commit subject, and the PR URL.

## Notes

- If `gh` isn't authenticated or there's no GitHub remote, do steps 1–4 and tell
  the user what's blocking the PR rather than failing silently.
- One logical change per PR. If the diff spans clearly unrelated changes, flag it
  and ask whether to split before committing.
- Don't amend or force-push existing history unless the user asks.
- Don't put lifecycle (`claude-*`) or `repo:` labels on the PR. Those are an
  issue-only concern; a PR's phase is its native Draft / Ready / Merged state.
