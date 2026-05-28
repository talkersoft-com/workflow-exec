# Deck: hive-deck-pro (via cloud-manager planning)

## Deck name
`hive-deck-pro`

## Repos with code changes

| Repo | What changes |
|------|-------------|
| `hive-deck-pro` | `internal/config/config.go`, `ops/ship.go`, `ops/await_merge.go` (new), `ops/contract.go`, `ops/contract/CONTRACT.md`, `ops/contract/schema.json`, `ops/dispatch.go`, `mcp/src/tools/await_merge.ts` (new), MCP rebuild |

## Branch
`dewy-bulldog` (cloud-manager planning) / current hive-deck-pro feature branch

## Initialize (Task 0000)

```
hv_status  deck: "hive-deck-pro"
```

## Ship code (hive-deck-pro deck)

```
hv_ship  deck: "hive-deck-pro"
         message: "feat: pr_mode enum — replace auto_merge + require_merged_pr booleans"
         title:   "feat: pr_mode enum — replace auto_merge + require_merged_pr booleans"
```

## Ship planning (cloud-manager deck)

```
hv_ship  deck: "cloud-manager"
         message: "planning: cloud-manager/dewy-bulldog"
         title:   "planning: cloud-manager/dewy-bulldog"
```
