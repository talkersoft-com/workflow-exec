# Semantic decorations — Epic plan 8 (ansible-p8-decorations)

## What this workflow does
Drives editor decorations from DB metadata: secret/required/vaultRef badges on `{{ var }}`
occurrences (C-VAR classification), hover provenance showing effective value + winner +
overridden chain for a chosen operation context (C-RES, secrets always `***`), and a
masking-everywhere sweep (`<SecretValue>`, ref-only vars editors, run pages on the masked
`rsnap` snapshot). Pure consumer of P4/P5/P7 contracts via the editor's extension points.
Runs under `feature@cloud-manager`.

## Read before starting
- `deck.md`, `../../../workflow-plans/cloud-manager/ansible-p8-decorations/PLAN.md`
- `../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` — §7 C-VAR, §8 C-RES,
  §9 C-ED: this plan consumes all three verbatim and may not reinterpret any of them
- `../../../workflow-plans/cloud-manager/ansible-p4-variables/PLAN.md`,
  `../../../workflow-plans/cloud-manager/ansible-p5-resolution/PLAN.md`,
  `../../../workflow-plans/cloud-manager/ansible-p7-editor/PLAN.md` — the dependency surface
- `../../../workflow-plans/master-plans/ANSIBLE-EPIC.md` — epic §8 (and §9 secret
  indicators, §10 masking)

## Constraints
- **Series order**: requires P4, P5, AND P7 merged — all three contracts must exist.
  Task 0000 verifies this before any code change.
- Consumes contracts as-is: no editor-component changes (C-ED extension points only), no API
  changes, no reinterpretation of masking — the client never receives secret cleartext and
  never tries to.
- Decorations are metadata-driven, never cosmetic guesses: no name-pattern heuristics
  (`*_token` etc.) — classification comes from C-VAR records only.
- Graceful degradation: pages must work with empty registry / no context selected.
- Code changes in cloud-manager-web ONLY.
