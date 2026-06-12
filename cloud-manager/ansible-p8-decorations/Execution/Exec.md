# Execution: cloud-manager/ansible-p8-decorations

## Objective
`cloud-manager-web` highlighting becomes *meaningful*: a new `useVarDecorations(documentContext)`
hook fetches C-VAR definitions for the document's owner closure (one batched call, cached per
revision), scans the P7 editor's parse tree for Jinja2 identifier nodes, and emits
`EditorDecoration[]` in the C-ED shape — `kind: "secret"` (lock badge + secret color token),
`"required"` (badge; bold gutter dot when resolution reports `missingRequired`), and
`"vaultRef"` (vault icon on `$secretRef` values and `vault_*` lookups) — with unregistered
variables left undecorated (a quiet tooltip hint only, no name-pattern heuristics ever). A
`hoverProvider` implementation calls the C-RES preview endpoint for the page's operation
context (debounced, per-context cached) and renders winner, overridden chain, and
`effectiveValue` — secrets arrive as the engine's literal `"***"` plus `sref_…` provenance;
cleartext never reaches the client. Editor pages gain an optional context bar (inventory +
host pickers, last context remembered per playbook via localStorage); without a context, hover
shows classification + definition info only and no resolution call is made. Masking becomes
systematic: a shared `<SecretValue>` component is the only way any component renders a var
value (greppable rule); vars editors render secret-classified values as `***` with a lock chip
and a "set via vault ref" affordance that writes the `$secretRef` form, never a value;
`RunDetailPage` shows lock chips on published-secret paths and the run vars panel renders from
the masked-by-construction `rsnap` instead of raw `varsSnapshot`. `useVarDecorations`,
`<SecretValue>`, and the context bar are exported as the **C-DECO** surface P9 consumes.
Decorations re-scan live (debounced 300ms, matching lint); pages degrade gracefully with an
empty registry or no context. All work in cloud-manager-web only — the P7 editor component,
the API, and the MCP are untouched. Web built and deployed; shipped on one exec-stage PR set.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/ansible-p8-decorations/PLAN.md` — the design and C-DECO surface
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` — §7 C-VAR, §8 C-RES, §9 C-ED (all consumed verbatim)
- `../../../../workflow-plans/cloud-manager/ansible-p4-variables/PLAN.md`, `../../../../workflow-plans/cloud-manager/ansible-p5-resolution/PLAN.md`, `../../../../workflow-plans/cloud-manager/ansible-p7-editor/PLAN.md` — the dependency surface
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC.md` — epic §8, §9 secret indicators, §10 masking
- `cloud-manager-web/src/components/AnsibleYamlEditor/` — the C-ED extension points being consumed (read-only)

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; verify P4/P5/P7 merged; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: metadata pipeline — C-VAR batched fetch + per-revision cache, identifier scan over the P7 parse tree, `EditorDecoration[]` emission
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: editor decorations — secret/required/vaultRef badges + gutter markers via C-ED props; theme tokens correct under both themes
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: hover provenance — context bar, debounced C-RES preview calls, provenance tooltip (winner / overridden chain / masked value)
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: masking sweep — `<SecretValue>`, vars-editor ref-only affordance, run pages on `rsnap`, lock indicators
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: regression + polish — no-context and unregistered-var behavior, hover cache perf, graceful degradation without P4/P5 data; export C-DECO surface
- [ ] `Tasks/0006-TASK.md` — **Final phase**: deploy web → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p8-decorations"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
