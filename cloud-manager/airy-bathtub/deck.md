# Deck: cloud-manager

## Execution branch
`stoneish-milkfish` — created by Task 0000 (hv_next from origin/hive) on 2026-06-27.

> The scaffold was generated for deck `cloud-manager`, orchestration `airy-bathtub`.
> The git execution branch is created in Task 0000, not pre-set.

## Hive Deck MCP calls

All deck git operations go through the hive-deck MCP — never raw `git` for commit / push / PR.

| Operation | Call |
|-----------|------|
| Status | `hv_status  deck: "cloud-manager"` |
| New branch (on a feature branch, PRs merged) | `hv_next  deck: "cloud-manager"` |
| New branch (on the default branch) | `hv_init  deck: "cloud-manager"` |
| Integrate (commit + push + PRs) | `hv_integrate  deck: "cloud-manager"  message: "..."  title: "..."  body: "..."` |
| Stash / restore (deadlock escape) | `hv_stash` / `hv_unstash  deck: "cloud-manager"` |
| Begin autonomous run | `hv_orchestrate_run  deck: "cloud-manager"  branch: "airy-bathtub"` |

## Repos in scope for this orchestration
| Repo | Node | Change |
|------|------|--------|
| `vorch-lib` | `vm-infra/cloud-manager` | cloud-init user-data ufw baseline |
| `cloud-manager-api` | `vm-infra/cloud-manager` | seed playbook YAMLs (`seed/playbooks/*.yaml`) |

Both repos are in the deck, so `hv_integrate` covers them. **No manual git steps required.**

## PR merge mode
PRs are queued for **auto-merge** on GitHub. `hv_integrate` detects the merges and
auto-transitions all repos to the next branch. The output may omit PR lines — verify open /
merged PRs with `gh pr list` / `hv_list_pulls` if the transition does not confirm.

## Build & deploy (see Exec.md `## Build` / `## Deploy`)
- Build: `python3 cloud-manager-mcp/.cicd/build-cloud-manager.py --target <api|mcp|web|all>`
- Deploy: `python3 cloud-manager-mcp/.cicd/deploy-cloud-manager.py --target <api|mcp|web|all>`
- vorch is built + redeployed manually on the hypervisor (Phase 0003 records the exact cmd).
