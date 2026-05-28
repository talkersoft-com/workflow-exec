# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes

| Repo | What changes |
|------|-------------|
| `workflow-exec` | Planning output files (this workflow scaffold) |

> Note: The actual code change is in `hive-deck-pro` repo which is on the `hive-deck-pro` deck. After writing the workflow files here, make the TypeScript change and ship via the `hive-deck-pro` deck separately.

## Branch
`icy-omelette`

## Initialize (Task 0000)

```
hv_status  deck: "cloud-manager"
```
Already on `icy-omelette` — no hv_next needed.

## Ship (planning files)

```
hv_ship  deck: "cloud-manager"
         message: "planning: cloud-manager/icy-omelette"
         title:   "planning: cloud-manager/icy-omelette"
```

## Ship (code — hive-deck-pro deck)

```
hv_ship  deck: "hive-deck-pro"
         message: "hv_workflow: replace plan_path with plan_paths array"
         title:   "hv_workflow: replace plan_path with plan_paths array"
```
