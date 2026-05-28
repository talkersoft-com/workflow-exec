# Test 0001 — Struct Changes

## TC-001 — WorkflowDef has Repo field
Run: `grep -n 'Repo' /path/to/hive-deck-pro/internal/config/config.go`
Pass: output contains `Repo string` inside the `WorkflowDef` struct.
Fail: no match found.

## TC-002 — DeckFile has no WorkflowFolder field
Run: `grep -n 'WorkflowFolder\|workflow_folder' /path/to/hive-deck-pro/internal/config/config.go`
Pass: zero matches (or only matches inside comments, not struct field definitions).
Fail: any live struct field definition found.

## TC-003 — Clean build
Run: `go build ./...` from the hive-deck-pro module root.
Pass: exits 0, no output.
Fail: any compile error.
