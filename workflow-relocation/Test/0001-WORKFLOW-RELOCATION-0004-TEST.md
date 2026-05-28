# Test 0004 — Config Files Updated

## TC-001 — workflows.yaml has repo field
Run: `grep 'repo:' ~/.hv/workflows.yaml`
Pass: output contains `repo: talkersoft-com/execution-workflow`.
Fail: no match.

## TC-002 — deck yaml has no workflow_folder
Run: `grep 'workflow_folder' ~/.hv/decks/cloud-manager.yaml`
Pass: no output (exit 1 from grep is a pass here).
Fail: any match found.

## TC-003 — hv workflow runs end-to-end
Run: `hv workflow cloud-manager`
Pass: exits 0, output contains the workflow scaffold with `{{WorkflowFolder}}` resolved to the correct on-disk path, no unreplaced tokens.
Fail: error, unreplaced `{{WorkflowFolder}}` token, or wrong path.

## TC-004 — Repo copy in sync
Run: `diff ~/.hv/workflows.yaml /path/to/hive-deck-pro/.hv/workflows.yaml`
Pass: no diff (files are identical).
Fail: files differ.
