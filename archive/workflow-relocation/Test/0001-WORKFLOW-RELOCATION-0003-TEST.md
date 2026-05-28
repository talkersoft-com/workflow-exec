# Test 0003 — Validation Rules

## TC-001 — Clean build
Run: `go build ./...` from the hive-deck-pro module root.
Pass: exits 0, no output.
Fail: any compile error.

## TC-002 — Valid deck passes validation
Run validation against the cloud-manager deck (with `repo: talkersoft-com/execution-workflow` present in `workflows.yaml`).
Pass: validation exits 0.
Fail: any validation error.

## TC-003 — Missing `repo` field fails validation
Temporarily remove the `repo:` line from the `cloud-manager` entry in `workflows.yaml`, run validation.
Pass: exits non-zero with message containing `repo is required`.
Fail: validation passes or wrong error message.
Restore `workflows.yaml` after this test.

## TC-004 — Repo not in deck fails validation
Temporarily set `repo: talkersoft-com/nonexistent` in `workflows.yaml`, run validation.
Pass: exits non-zero with message containing `is not in this deck`.
Fail: validation passes or wrong error message.
Restore `workflows.yaml` after this test.
