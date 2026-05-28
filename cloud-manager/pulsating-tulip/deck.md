# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `execution-workflow` | workflow files for this task |

## Branch
`pulsating-tulip`

## Initialize (Task 0000)
Deck already on `pulsating-tulip`. Verify:
```
hv_status  deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "align claude_settings to Claude native schema, disable bypassPermissions"
         title:   "config: align claude_settings permissions shape to Claude schema"
```

## Post-ship init
```
hv_init  deck: "cloud-manager"
```

## Manual steps (hive-deck-pro is outside the cloud-manager deck)
```
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro add \
    internal/config/config.go \
    internal/claude/claude.go \
    .hv/config.yaml.example
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro commit -m "config: align claude_settings permissions shape to Claude schema"
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro push origin HEAD
```
