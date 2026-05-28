# Deck: cloud-manager + hive-deck-pro

## Repos with code changes

| Repo | What changes |
|------|-------------|
| `hive-deck-pro` | `internal/config/config.go` (HV_HOME search), `Makefile`, `scripts/configure-hv-home.sh` (new) |
| `workflow-configuration` | New repo — seeded with current `~/.hv/` contents |
| `cloud-manager` deck config | `cloud-manager.yaml` — add workflow-configuration to config node |

## Branch
`drizzly-crow` (cloud-manager planning) / current hive-deck-pro feature branch

## Initialize (Task 0000)

```
hv_status  deck: "cloud-manager"
hv_status  deck: "hive-deck-pro"
```

## Ship code (hive-deck-pro deck)

```
hv_ship  deck: "hive-deck-pro"
         message: "feat: HV_HOME direct path support + make configure script"
         title:   "feat: HV_HOME direct path support + make configure"
```

## Ship planning (cloud-manager deck)

```
hv_ship  deck: "cloud-manager"
         message: "planning: cloud-manager/drizzly-crow"
         title:   "planning: cloud-manager/drizzly-crow"
```
