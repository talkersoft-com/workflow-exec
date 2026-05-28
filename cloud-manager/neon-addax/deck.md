# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | Migration + DTOs + services + controllers + AutoMapper profiles + DI registrations per API-PLAN.md |
| `execution-workflow` | `Results/RESULT.md` + `Retro/LESSONS.md` inside this workflow folder (per project convention) |

Task 0011's `hv_ship` therefore produces **two PRs simultaneously** (one per
dirty repo). All other deck repos are clean and skipped. The scaffold itself
ships in its own PR before `/loop` runs — see `init.md` for the full
lifecycle.

## Branch
`cosmic-strudel` — recorded by Task 0000 from `hv_status` output. The workflow
folder name `neon-addax` is the workflow ID, not the branch name.

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
```
- If all 14 repos are clean and on the **same feature branch**: record the
  branch name in this file's `## Branch` section and proceed.
- If all 14 repos are clean and on the **default branch**: run
  `hv_next  deck: "cloud-manager"`, then re-run `hv_status` and record the
  generated branch name.
- If any repo is dirty: STOP. Investigate before proceeding.

## Ship
```
hv_ship  deck: "cloud-manager"
         message: "feat(api): implement ansible REST surface (cloud-manager-ansible/API-PLAN.md)"
         title:   "Implement ansible REST surface in cloud-manager-api"
```
