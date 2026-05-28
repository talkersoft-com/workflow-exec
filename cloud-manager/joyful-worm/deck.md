# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `hive-deck-pro` | Makefile `install-config` target — safe-by-default copy with `FORCE=1` override |

## Branch
`cobalt-opossum`

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
         message: "make install-config: safe by default, FORCE=1 to overwrite"
         title:   "install-config: never silently overwrite local ~/.hv edits"
```
