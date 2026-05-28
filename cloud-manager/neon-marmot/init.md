# force_overwrite for claude_settings

## What this workflow does
Adds a `force_overwrite` boolean field to the `ClaudeSettings` config struct in hive-deck-pro. When `true` (the default), `MaybeWrite` replaces any existing `settings.local.json` with the full permission set from config rather than additively merging. When `false`, the legacy additive-merge behavior is preserved. This ensures that removing a permission from `config.yaml` is reflected in all repos on the next `hv init`, without requiring manual cleanup.

## Read before starting
- `deck.md` — which repos are in scope; hv MCP calls and manual git steps
- `Orchestrate/ORCH.md` — full task list

## Constraints
- Changes are confined to hive-deck-pro: `internal/config/config.go`, `internal/claude/claude.go`, and `.hv/config.yaml.example`
- No changes to any other subsystem
- hive-deck-pro is outside the cloud-manager deck — all commits must be done manually per `deck.md`
