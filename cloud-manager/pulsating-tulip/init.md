# Align claude_settings YAML to Claude's native schema

## What this workflow does
Restructures the `claude_settings` block in hive-deck-pro so the subtree beneath
`permissions:` maps 1:1 to Claude Code's own `settings.local.json` schema. This removes
the custom flat shape (`allow:`, `default_mode:` at the top level) and replaces it with
a `permissions:` node whose fields (`allow`, `deny`, `defaultMode`) serialise directly
to/from Claude's JSON format with no translation layer. `bypassPermissions` is turned off
by leaving `defaultMode` commented out.

## Read before starting
- `deck.md` — repos in scope; manual git steps for hive-deck-pro
- `Orchestrate/ORCH.md` — task list

## Constraints
- Only `ClaudeSettings` and `claude_settings` are touched — no other structs or config keys
- hive-deck-pro is outside the cloud-manager deck — all commits are manual
