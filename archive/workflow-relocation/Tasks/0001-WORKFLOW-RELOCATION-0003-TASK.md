# Task 0003 — Add validation in `validate.go`

## Goal
Add a validation rule that checks the workflow's `repo` field: it must be non-empty, and the repo it names must exist in the deck's repo list.

## Target repo(s)
`hive-deck-pro`

## Steps
1. Open `internal/config/validate.go`
2. In the deck validation pass, after loading the workflow definition:
   a. If `wfDef.Repo == ""`: return error `workflow "<name>": repo is required`
   b. Collect all repo names from the deck's resolved node tree
   c. If `wfDef.Repo` is not in that list: return error `workflow "<name>": repo "<repo>" is not in this deck`
3. Run `go build ./...` — fix any compile errors

## Definition of done
`go build ./...` passes. `hv validate cloud-manager` (or equivalent validation entrypoint) passes for a valid deck and fails with a clear message for a deck with a missing or mismatched workflow repo.
