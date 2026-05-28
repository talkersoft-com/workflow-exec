# Remove wildcard permission + version bump

## What this workflow does
Removes the bare `"*"` wildcard from `claude_settings.allow` in both the hive-deck-pro
source (`config.yaml.example`) and the live user config (`~/.hv/config.yaml`). Also bumps
the MCP package version from `0.1.0` to `0.1.1`. After changes are committed and pushed,
the deck is reinitialised so all repos immediately get fresh `settings.local.json` files
without the wildcard.

## Read before starting
- `deck.md` — repos in scope; hv MCP calls and manual git steps for hive-deck-pro
- `Orchestrate/ORCH.md` — full task list

## Constraints
- Only `"*"` is removed from the allow list — all explicit tool entries stay
- No changes to any Go source; only `config.yaml.example`, `mcp/package.json`, and
  `~/.hv/config.yaml` are touched
- hive-deck-pro is outside the cloud-manager deck — all commits are manual
