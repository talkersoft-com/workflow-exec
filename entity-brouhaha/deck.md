# Deck Manifest: entity-brouhaha

## Deck

`cloud-manager`

## Generated branch name

leafy-sable

## PR URL / merge commit

*(filled in after `hv_ship` completes)*

## Repos with code changes in this workflow

| Repo | Changes |
|---|---|
| `cloud-manager-api` | Entity classes, DbContext config, DTO cleanup, service cleanup, EF migration |

All other repos in the `cloud-manager` deck will be on the feature branch but will have no changes. `hv_ship` skips them automatically.

## How to initialize (Task 0000)

```
hv_status deck: "cloud-manager"   ← verify workspace is clean
hv_init   deck: "cloud-manager"   ← provisions repos + creates feature branch
```

If the workspace is already provisioned on a prior feature branch with all PRs merged:
```
hv_status deck: "cloud-manager"
hv_next   deck: "cloud-manager"
```

## How to ship (Task 0006)

```
hv_ship deck: "cloud-manager"
  message: "feat: playbook management data model — EF entities, migration, remove git-backed code"
  title: "entity-brouhaha: playbook management data model"
```
