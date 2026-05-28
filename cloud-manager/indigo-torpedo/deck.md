# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `planning` | 5 new markdown files under `planning/cloud-manager-ansible/` |
| `execution-workflow` | `Results/RESULT.md` + `Retro/LESSONS.md` inside this workflow folder (per project convention) |

Task 0006's `hv_ship` therefore produces **two PRs simultaneously** (one per
dirty repo). All other deck repos are clean and skipped. The scaffold itself
was already shipped on prior branches — see `init.md` for the full lifecycle.

## Branch
`fearless-heron` — recorded by Task 0000 from `hv_status` output. The workflow
folder name `indigo-torpedo` is the workflow ID, not the branch name.

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
         message: "plan: ansible-management bundle for api, web, vorch (cloud-manager-ansible/)"
         title:   "Add cloud-manager-ansible planning bundle"
```
