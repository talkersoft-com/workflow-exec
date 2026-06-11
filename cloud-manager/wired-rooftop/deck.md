# Deck: cloud-manager

## Deck name
`cloud-manager`

## Orchestration id (plan codename)
`wired-rooftop`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-web` | BlueprintsPage row Edit affordance; create-modal success navigation |
| `workflow-exec` | This orchestration folder (runtime) |

**Must NOT change:** everything else.

## Branch
TBD — created by Task 0000 at execution time; record the generated name here.

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "fix: blueprint composer navigation — list edit affordance + create lands on composer"
         title:   "Blueprint composer navigation (wired-rooftop)"
         stage:   "exec"
```
