# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `hive-deck-pro` | `config.yaml.example` — comment out `- "Bash(*)"` from `claude_settings.allow` |

## Branch
`velour-birch`

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_init    deck: "cloud-manager"
```
If already provisioned on a prior feature branch with all PRs merged:
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "remove Bash(*) from claude_settings.allow in config.yaml.example"
         title:   "config: remove unrestricted Bash access from deck repo settings"
```

## Manual steps (hive-deck-pro is outside the cloud-manager deck)
hive-deck-pro changes must be committed and pushed manually on `warm-leviathan`:
```
git -C /path/to/hive-deck-pro add .hv/config.yaml.example
git -C /path/to/hive-deck-pro commit -m "remove Bash(*) from claude_settings.allow"
git -C /path/to/hive-deck-pro push origin HEAD
```
