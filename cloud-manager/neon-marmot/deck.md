# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `execution-workflow` | workflow files for this task (init.md, deck.md, tasks, tests) |

All other deck repos are on the feature branch but have no changes — hv_ship will skip them.

## Branch
`neon-marmot`

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```
(deck was already provisioned on neon-marmot by the user before starting)

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "add force_overwrite workflow files"
         title:   "workflow: add force_overwrite for claude_settings"
```

## Manual steps (hive-deck-pro is outside the cloud-manager deck)
All Go and config changes to hive-deck-pro must be committed and pushed manually on `warm-leviathan`:
```
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro add \
    internal/config/config.go \
    internal/claude/claude.go \
    .hv/config.yaml.example
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro commit -m "feat: add force_overwrite to claude_settings (default true)"
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro push origin HEAD
```
