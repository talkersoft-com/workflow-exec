# Task 0004 — Update config files

## Goal
Update all YAML config files to match the new schema: add `repo:` to workflow definitions and remove `workflow_folder` from deck files.

## Target repo(s)
`hive-deck-pro` (for `.hv/` repo copies)

## Steps
1. Update `~/.hv/workflows.yaml` — add `repo: talkersoft-com/execution-workflow` under the `cloud-manager:` entry
2. Update `/path/to/hive-deck-pro/.hv/workflows.yaml` — same change (repo copy)
3. Update `~/.hv/decks/cloud-manager.yaml` — remove the `workflow_folder: execution-workflow` line
4. Update `/path/to/hive-deck-pro/.hv/decks/cloud-manager.yaml` — same change (repo copy)
5. Update `~/.hv/deck.yaml.example` and the repo copy — remove `workflow_folder` if present, ensure the example reflects the new pattern
6. Run `hv workflow cloud-manager` to confirm end-to-end output is correct

## Definition of done
`hv workflow cloud-manager` runs without error, `{{WorkflowFolder}}` resolves correctly, and no YAML file contains `workflow_folder`.
