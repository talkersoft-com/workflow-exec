# Task 0001 — Add `Repo` to `WorkflowDef`, remove `WorkflowFolder` from `DeckFile`

## Goal
Update the config structs so the workflow definition carries the repo reference and the deck file no longer carries `workflow_folder`.

## Target repo(s)
`hive-deck-pro`

## Steps
1. Open `internal/config/config.go`
2. Add `Repo string \`yaml:"repo"\`` to the `WorkflowDef` struct
3. Remove `WorkflowFolder string` field from the `DeckFile` struct (yaml key `workflow_folder`)
4. Run `go build ./...` from the module root — fix any compile errors before continuing

## Definition of done
`go build ./...` passes. `WorkflowDef` has a `Repo` field. `DeckFile` has no `WorkflowFolder` field.
