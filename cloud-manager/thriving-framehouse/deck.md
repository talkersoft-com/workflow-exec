# Deck: cloud-manager

## Orchestration
- **Deck:** `cloud-manager`
- **Orchestration:** `thriving-framehouse`
- **Workflow ref:** `feature@cloud-manager`
- **Branch:** `fuchsia-deertrail` — recorded by Task 0000 (created via hv_next from origin/hive, 2026-06-26)
- **Workflow folder:** `/home/todd/workspace/cloud-manager/planning/workflow-exec/cloud-manager/thriving-framehouse`

## Deck operations (Hive Deck MCP)
Use these MCP tools for all deck git operations. Never use raw `git` to commit, push, or open PRs
for repos managed by the deck.

| Tool | Call | Purpose |
|------|------|---------|
| `hv_status` | `deck: "cloud-manager"` | Report git state of every repo. Run first in Task 0000. |
| `hv_init` | `deck: "cloud-manager"` | Provision missing repos + branch off `origin/<default>`. Use when on default branch. |
| `hv_next` | `deck: "cloud-manager"` | Transition all repos to a new branch off `origin/hive`. Use on a feature branch with PRs merged. |
| `hv_integrate` | `deck: "cloud-manager"  message: …  title: …  body: …` | Commit + push + open PRs for every repo in one step. **No `stage` param.** |
| `hv_stash` / `hv_unstash` | `deck: "cloud-manager"` | Stash/restore across all repos — deadlock escape hatch. |
| `hv_orchestrate_run` | `deck: "cloud-manager"  branch: "thriving-framehouse"` | Begin autonomous execution from Exec.md. |

## Repos affected by this orchestration
- `cloud-manager-api` (.NET) — migration + provision populate + read endpoint.
- `cloud-manager-mcp` (TypeScript) — camelCase guard exemption.

All affected repos are in the deck — no `## Manual steps` section required.

## Notes
- PRs auto-merge on GitHub; `hv_integrate` detects merges and auto-transitions the deck.
- Restart `cloud-manager-mcp` via `/mcp` after a merged PR that changes its dist (new/renamed
  tools or changed parameters). This change is additive to argument validation — `/mcp` reload
  still required for the running server to accept the previously-rejected calls.
