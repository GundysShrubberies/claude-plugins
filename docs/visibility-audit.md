# Visibility audit: taking the repo private

Part of GundysShrubberies/arbor#1198 (plan fork F2: all proprietary, private).
Rule: fix each consumer's auth before the visibility flip, never after.

## Consumers that read this repo

Found in the repo itself: the marketplace add command in `README.md` and
`plugins/gundy-kit/README.md`, and `.claude-plugin/marketplace.json`. Anything
else is listed from the fleet's setup, not from a probe of each host.

| Consumer | Reads how | Breaks when private? | Auth fix before the flip |
| --- | --- | --- | --- |
| `/plugin marketplace add GundysShrubberies/claude-plugins` (README, plugin README) | git clone as the user | Only without a GitHub login | Needs `gh auth login` (already a stated requirement) |
| Plugin auto-update at session start | git fetch, background | Yes, silently, unless a token is in the env | Export `GITHUB_TOKEN` (read scope on this repo) where auto-update must run |
| Cloud routines | Anthropic-hosted clone | Yes, unless the routine's GitHub access includes this repo | Grant the Claude GitHub app access to this repo |
| The Mac | git clone as the user | Only without `gh auth` | Run `gh auth status`; log in if it fails |
| Second Claude account (`~/.claude-b`) | same GitHub login | No, shares the GitHub login | `gh auth status` under that profile |
| Fleet workers and timers | `arbor-gh` app token | No | The `claude-plugins` app token already exists |

## Not verified here

A worker cannot reach the Mac, the second profile's credentials or the cloud
routine settings. Each row above is a check for the operator seat to run before
the flip, not a result.

## Decision

The repo can go private once the three auth fixes above are confirmed. No
consumer needs it public, so no reason to stay public is recorded. The LICENSE
and the README copyright line apply either way.
