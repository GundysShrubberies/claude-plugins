# gundy-kit

A Claude Code plugin: slash commands, skills, scripts, and hooks pulled out of
one person's day-to-day setup and generalized for shared use. No secrets, no
private infra — everything here assumes only a normal GitHub account and `gh`.

## Install

```
/plugin marketplace add GundysShrubberies/claude-plugins
/plugin install gundy-kit@gundy-kit
```

You'll need `gh` (the GitHub CLI) installed and authenticated (`gh auth
login`) for anything that touches GitHub: `/commit-push-pr`, `/handoff`,
`/pickup`, and the PR-checks-waiting script. `jq` is required by every script
here; if it's missing, the affected script/hook fails open (prints a message,
or does nothing) rather than breaking your session.

## What's in it

### Commands

- **`/harden`** — pressure-tests a plan or decision before you commit to it:
  hidden assumptions, failure modes, a simpler path. Says "solid, go" when
  there's nothing to add — it doesn't manufacture concerns to look thorough.
- **`/prototype`** — generates several genuinely different self-contained HTML
  prototypes of an idea (a page, a component) so you can compare directions
  side by side, and recommends one.
- **`/commit-push-pr`** — takes the working tree from `git status` through a
  committed, pushed, opened pull request in one flow, matching your repo's
  commit style and linking an issue if it can find one.
- **`/cloud-agents`** — lists your scheduled cloud agents (claude.ai
  "routines"): schedule, model, enabled state, and where to manage each.
  Read-only.
- **`/handoff [slug]`** — writes a summary of the current session's
  in-progress work to a shared GitHub-backed store, so anyone (or you, on a
  different machine) can pick it up later.
- **`/pickup [slug]`** — lists or claims a handoff from that store. Claiming
  archives it immediately, so two people can't both pick up the same one.

### Skills

- **`herdr`** — drives the [Herdr](https://herdr.dev) terminal multiplexer for
  coding agents, if you use it. Only activates when you mention Herdr and
  `HERDR_ENV=1` is set; otherwise it's inert.
- **`check-buffer`** — reviews whatever file you currently have open in nvim,
  without you having to type the path. Needs a small autocmd in your
  `init.lua` — the skill's own file has the snippet and says where to put it.

### Scripts (`scripts/`)

These back the commands and hooks above; you generally don't call them
directly, but they're plain, documented, `shellcheck`-clean bash if you want
to:

- `claude-work-item` — tracks "what issue/phase is this session on" for the
  status line, and logs+optionally-announces when a work item finishes.
- `claude-context-gauge` — reads a session transcript and reports how full
  the live context window is.
- `claude-statusline` — the actual status-line renderer (repo/branch or
  active work item, plus a color-coded context-window percentage).
- `pr-checks-wait` — blocks in one foreground call until every CI check on a
  PR's head commit settles, instead of polling turn-by-turn. Understands
  rate limits, a moved head SHA, and a wedged/offline self-hosted runner.
- `claude-tool-guard` — the `PreToolUse` hook (see below) that denies
  wasteful tool-call shapes.
- `claude-handoff` — writes/reads the local session hand-off file used by the
  context-window guard, and pushes/pulls it to the shared store via
  `kit-handoff`.
- `kit-handoff` — the actual GitHub-Contents-API client for the shared
  hand-off store (list/write/read/pickup/archive/template).
- `kit-session-init`, `kit-working-style` — small SessionStart hook helpers
  (see below).

### Hooks (`hooks/hooks.json`)

- **`SessionStart`** — on every session start (and `/clear`):
  1. `kit-session-init` records this plugin's installed path to
     `~/.claude/gundy-kit/plugin-root`, so commands that need to invoke a
     script can find it (see "Why commands read a file to find their own
     scripts" below).
  2. `claude-handoff inbox` prints the shared hand-off store's open items, if
     any, so you notice one waiting for you.
  3. `kit-working-style` prints two standing working-style instructions
     ("always recommend an option", "delegate investigation to agents").
- **`PreToolUse`** (matcher `Bash|Monitor|Agent`) — `claude-tool-guard` denies
  a handful of tool-call shapes that reliably waste turns: no-op commands
  used to pass time, blocking CI watchers (`gh run watch`, `gh pr checks
  --watch`), a CI wait backgrounded instead of run in the foreground, a
  subagent spawning another subagent, and (for subagents specifically) CI
  status polling, long sleeps, and the same command repeated in a loop. It
  also has an optional context-window guard: past a configurable percentage
  it injects a reminder to wrap up and write a hand-off; past a higher one
  for a subagent, it denies the call outright and asks for a partial-status
  return. Every threshold is overridable via environment variables — see the
  comment at the top of `scripts/claude-tool-guard`.

## Why commands read a file to find their own scripts

Claude Code substitutes `${CLAUDE_PLUGIN_ROOT}` inside hook commands, MCP/LSP
server config, and skill/agent content — but **not** inside a command's own
Markdown body (checked against
[the plugins reference](https://code.claude.com/docs/en/plugins-reference.md)).
A plugin's `scripts/` directory also isn't added to `PATH` automatically —
only a top-level `bin/` gets that treatment, and this plugin deliberately
doesn't use one (executables live in `scripts/`, per this repo's layout).

So `/handoff` and `/pickup`, which need to run `scripts/kit-handoff`, can't
just reference `${CLAUDE_PLUGIN_ROOT}/scripts/kit-handoff` in their own text
and have it resolve. Instead, the `kit-session-init` SessionStart hook — which
*does* get `${CLAUDE_PLUGIN_ROOT}` as an environment variable — writes it to
`~/.claude/gundy-kit/plugin-root` on every session start. The two commands
read that file to find their scripts. If it's missing (the plugin was just
installed mid-session, before any hook ran), run `/clear` once, or find the
scripts by hand: `find ~/.claude -maxdepth 6 -name kit-handoff -type f`.

## Setting the status line

Plugins can't set `statusLine` for you — it's a top-level field in your own
`~/.claude/settings.json`, and nothing in a plugin manifest can reach it. To
use `claude-statusline`, add this yourself once you know the plugin's
installed path (`cat ~/.claude/gundy-kit/plugin-root` after your first
session with the plugin enabled):

```json
{
  "statusLine": {
    "type": "command",
    "command": "<plugin-root>/scripts/claude-statusline"
  }
}
```

## Disabling

`/plugin uninstall gundy-kit@gundy-kit` removes it. To disable temporarily
without uninstalling, use `/plugin` and toggle it off from there. Either way,
if you added a manual `statusLine` entry per the section above, remove that
line from `settings.json` too — a disabled plugin won't error if it's left in
place (the command just won't be found), but it'll also stop showing anything
useful.

## Board & vibe-coding kit

Everything below reads `.kit/config.json` at the root of your repo checkout:
`{"repo", "project_owner", "project_number", "merge_allowlist"}`. There are no
defaults. If the config is missing, the scripts stop and tell you why.

- **`/start-here`**: first-run walkthrough. Checks GitHub login, clones the
  workspace, sets up the board config, turns on guided mode, and walks you
  through your first idea.
- **`/vibe [what you want]`**: say what you want in your own words. Claude
  asks a couple of plain-English questions, reads the plan back to you, and
  queues it on the board.
- **`/raise-issue <desc>`**: the technical version. Drafts a well-formed issue
  and queues it in Approved.
- **`/status`**: what's being built, what's stuck, and what's done.
- **`/guided on|off|status`**: when on, every session explains each step in
  plain English.
- **`kit-setup --project-number N`**: one-time setup for a repo. Writes the
  config, creates the `kit-working` and `kit-needs-help` labels, and checks the
  board. It doesn't create the project board itself; ask Connor for that.
- **`kit-board add|move|list|check`**: board operations. The Status options are
  Backlog, Approved, In Progress, Needs help, and Done.
