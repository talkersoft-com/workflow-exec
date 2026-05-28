# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `execution-workflow` | workflow files for this task |

All other deck repos are on the feature branch but have no changes — hv_ship will skip them.

## Branch
`steamy-radish`

## Initialize (Task 0000)
Deck is already provisioned on `steamy-radish`. Just verify:
```
hv_status  deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "remove wildcard permission, bump mcp version to 0.1.1"
         title:   "config: remove * from claude_settings.allow, bump mcp to 0.1.1"
```

## Post-ship init
After hv_ship completes, all repos return to default branch. Reinitialise to apply
updated settings to all repos:
```
hv_init  deck: "cloud-manager"
```

## Manual steps (hive-deck-pro is outside the cloud-manager deck)
```
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro add \
    .hv/config.yaml.example \
    mcp/package.json
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro commit -m "config: remove * from claude_settings.allow, bump mcp to 0.1.1"
git -C /Users/talker/workspace/hive-deck-pro/hive-deck-pro push origin HEAD
```
