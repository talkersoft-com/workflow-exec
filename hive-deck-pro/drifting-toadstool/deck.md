# Deck: hive-deck-pro

## Deck name
`hive-deck-pro`

## Orchestration id (plan codename)
`drifting-toadstool`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `hive-deck-pro` | expandPath token engine, Setup.DeckConfig + layerPaths, per-file loader fallback, union scans (decks list, workflow registry, 3-step fragment chain), HV_PROFILE machine-identity selection, mcps token table, tests, README docs |
| `workflow-exec` | This orchestration folder (runtime updates) |

**Must NOT change:** workflow-configuration (base/ migration is a follow-up config change),
workflow-fragments, workflow-plans (beyond nothing — no findings docs in this plan).

## Branch
`fragrant-hideaway` — created by Task 0000 via hv_next on 2026-06-11.

## Initialize (Task 0000)
```
hv_status  deck: "hive-deck-pro"
hv_next    deck: "hive-deck-pro"
```
(or `hv_init` if repos are on the default branch)

## Ship
```
hv_ship  deck: "hive-deck-pro"
         message: "feat: shared config layer + canonical path tokens (R7.3-R7.6)"
         title:   "Shared config layer + WorkspaceRoot tokens (drifting-toadstool)"
         stage:   "exec"
```
