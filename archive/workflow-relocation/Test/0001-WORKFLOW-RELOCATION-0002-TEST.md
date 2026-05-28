# Test 0002 — workflowCmd Repo Resolution

## TC-001 — Clean build
Run: `go build ./...` from the hive-deck-pro module root.
Pass: exits 0, no output.
Fail: any compile error.

## TC-002 — WorkflowFolder token resolves to correct path
Run: `hv workflow cloud-manager`
Pass: output contains `/Users/talker/workspace/cloud-manager/planning/execution-workflow` (the on-disk path of `talkersoft-com/execution-workflow`).
Fail: path is wrong, token is unreplaced (`{{WorkflowFolder}}`), or command errors.

## TC-003 — Missing repo produces clear error
Temporarily add a workflow entry in `workflows.yaml` pointing to a repo not in the cloud-manager deck (e.g. `repo: talkersoft-com/nonexistent`), then run `hv workflow cloud-manager`.
Pass: exits non-zero with message containing `requires repo` and the repo name.
Fail: silent failure, wrong error, or panic.
Restore the original `workflows.yaml` after this test.
