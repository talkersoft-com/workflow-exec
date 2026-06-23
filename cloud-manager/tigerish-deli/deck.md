# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | New `seed/playbooks/rabbitmq-jammy.{yaml,meta.json}` + `rabbitmq-noble.{yaml,meta.json}`; new EF migration seeding both `rabbitmq-*` blueprints + blueprint_playbooks links. |

Repos not listed are on the feature branch but skipped by hv_ship.
`cloud-manager-mcp/.cicd/import-playbooks.py` reused unchanged (run at verify time).

## Branch
TBD — created by Task 0000 (hv_init/hv_next) and recorded here.

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
         message: "feat: RabbitMQ marketplace blueprint (jammy + noble)"
         title:   "feat: RabbitMQ marketplace blueprint (jammy + noble)"
         stage:   "exec"
```
