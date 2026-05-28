# Deck: hive-deck-pro (via cloud-manager planning)

## Deck name
`hive-deck-pro`

## Repos with code changes

| Repo | What changes |
|------|-------------|
| `hive-deck-pro` | `ops/` package, `cmd/hv/main.go` wiring, `mcp/src/runner.ts` + all tool call sites |

> Note: Code changes ship on the `hive-deck-pro` deck. The `cloud-manager` deck is used only for planning files (workflow-exec).

## Branch
`enormous-kookaburra` (cloud-manager planning branch)

## Initialize (Task 0000)

```
hv_status  deck: "hive-deck-pro"
```
hive-deck-pro should already be clean on its current feature branch.

## Ship code (hive-deck-pro deck)

```
hv_ship  deck: "hive-deck-pro"
         message: "refactor: ops/ package + JSON CLI dispatch + MCP runner update"
         title:   "refactor: ops/ package + JSON CLI dispatch + MCP runner update"
```

## Ship planning (cloud-manager deck)

```
hv_ship  deck: "cloud-manager"
         message: "planning: cloud-manager/enormous-kookaburra"
         title:   "planning: cloud-manager/enormous-kookaburra"
```
