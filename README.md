# claude-plugins

A private Claude Code plugin marketplace.

## Install

```
/plugin marketplace add GundysShrubberies/claude-plugins
/plugin install gundy-kit@gundy-kit
```

## What's here

- **`gundy-kit`** — commands, skills, scripts, and hooks for day-to-day
  Claude Code use: hardening a plan before committing to it, generating
  side-by-side HTML prototypes, a full commit→push→PR flow, listing
  scheduled cloud agents, cross-machine work hand-offs, and a hook that
  denies a handful of tool-call patterns known to waste turns. See
  [`plugins/gundy-kit/README.md`](plugins/gundy-kit/README.md) for the full
  breakdown of what each piece does and how to configure it.

## Layout

```
.claude-plugin/marketplace.json   — marketplace manifest
plugins/gundy-kit/
  .claude-plugin/plugin.json      — plugin manifest
  commands/                       — slash commands
  skills/                         — skills
  scripts/                        — executables the commands/hooks call
  hooks/hooks.json                — SessionStart + PreToolUse hooks
  README.md                       — plugin-specific docs
```

## Requirements

`gh` (authenticated) and `jq` for the pieces that touch GitHub or parse JSON.
Nothing here depends on any private infrastructure, secret, or account beyond
your own GitHub login.
