# Remove Bash Permissions from Deck Repos

## What this workflow does
Comments out `- "Bash(*)"` from `claude_settings.allow` in the hive-deck-pro source repo's `config.yaml.example`, deploys it via `make install-config FORCE=1`, then reinitializes the cloud-manager deck so every repo gets a fresh `.claude/settings.local.json` without unrestricted Bash access. After this, Claude cannot use raw git or shell commands in cloud-manager repos.

## Read before starting
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive

## Constraints
- Do not remove any other entries from `claude_settings.allow` — only `Bash(*)`
- `make install-config FORCE=1` will overwrite `~/.hv/config.yaml` — this is intentional
- After reinit, Claude will no longer have Bash access in cloud-manager repos
