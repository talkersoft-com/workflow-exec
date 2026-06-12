# Execution: cloud-manager/ansible-p7-editor

## Objective
`cloud-manager-web`'s plain `<textarea>` role-file editor is replaced by `AnsibleYamlEditor`,
a CodeMirror 6 component implementing contract **C-ED** (contracts §9) verbatim: YAML-structure
highlighting (keys/values, anchors/aliases, block scalars, comments, indent guides) via
`@codemirror/lang-yaml`; Ansible play/task keywords and module-name positions highlighted as a
styling overlay on the YAML tree; `{{ … }}` / `{% … %}` Jinja2 parsed as an embedded template
language by a small in-repo Lezer grammar (delimiters, identifiers, filters get their own
highlight tags); `fragmentMode` virtually wraps non-standalone chunks before parse and maps
positions back so highlighting/diagnostics never error, with relative indentation preserved on
save; a diagnostics surface (squiggles + gutter + lint panel) merging client-side YAML syntax
errors from the Lezer tree with caller-supplied `diagnostics`; one CM6 theme reading the
existing `--color-*` SCSS custom properties, correct under both `[data-theme]` values; and
inert `decorations` / `hoverProvider` extension points (compartment + hoverTooltip bridge)
that P8 plugs into without touching this plan's files. Existing props
(`initialContent`/`onSave`/`onDirtyChange`) keep exact semantics — dirty tracking, Cmd/Ctrl-S,
beforeunload prompt, and change-summary flow byte-identical in behavior; both consumer pages
(`AnsiblePlaybookEditorPage`, `AnsibleRoleDetailPage`) swapped; the editor buffer remains the
source of truth (no YAML re-serialization); all CM6 packages stay behind the existing
`React.lazy` boundary with a Vite build-size guard (±10%). Web built and deployed; shipped on
one exec-stage PR set. No changes outside cloud-manager-web.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/ansible-p7-editor/PLAN.md` — C-ED contract and extension-stack design
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` — §9 (C-ED is binding verbatim; P8 consumes it)
- `../../../../workflow-plans/master-plans/ANSIBLE-EPIC.md` — epic §7
- `cloud-manager-web/src/components/RoleFileEditor/RoleFileEditor.tsx` + both consumer pages — the API being preserved

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; record the execution branch in deck.md
- [x] `Tasks/0001-TASK.md` — **Phase 1**: CM6 foundation — packages, `AnsibleYamlEditor` with legacy props parity, theme from SCSS tokens, lazy-chunk budget
- [x] `Tasks/0002-TASK.md` — **Phase 2**: YAML + Ansible semantics — lang-yaml integration, keyword/module overlay, indent guides
- [x] `Tasks/0003-TASK.md` — **Phase 3**: embedded Jinja2 — in-repo Lezer template grammar, nested mounting in scalars, highlight tags
- [x] `Tasks/0004-TASK.md` — **Phase 4**: fragments + diagnostics — fragmentMode wrap/map, client parse lint, `diagnostics` prop merge, gutter/panel
- [x] `Tasks/0005-TASK.md` — **Phase 5**: extension points + regression — decorations/hoverProvider compartments (inert), both consumer pages swapped, dirty/save/beforeunload behavior byte-identical
- [ ] `Tasks/0006-TASK.md` — **Final phase**: deploy web → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p7-editor"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
