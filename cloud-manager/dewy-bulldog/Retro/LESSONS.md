# Lessons: cloud-manager/dewy-bulldog

## What would have helped

1. `AwaitMergeInput` declared in `await_merge.go` — no need to add it to `contract.go` separately. The pattern from earlier phases (declaring struct in the op file) should be the default.
2. `index.ts` couldn't be edited with the Edit tool after the file had been read via the Read tool in a prior session — using Bash sed was the workaround. Worth noting for future MCP index edits.
3. `go test ./...` and `go install` must be run from the module root, not a subdirectory — easy to accidentally `cd` into `mcp/`.
4. The backward-compat `UnmarshalYAML` pattern (decode raw map, inspect keys, derive enum) is clean and reusable for any future config migrations.

## Fix files written

None.
