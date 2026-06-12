# Result — ansible-p7-editor (Ansible Epic P7: real editor)

## Outcome
**Complete.** The plain `<textarea>` role-file editor is replaced by
`AnsibleYamlEditor`, a CodeMirror 6 component implementing contract **C-ED**
(contracts §9) verbatim. All six phases implemented, all task tests pass
(37 vitest component/unit tests), web built, deployed, and smoke-verified
through the deployed stack. One FIX (pre-existing `file_path`/camelCase wire
mismatch crashing `AnsibleRoleDetailPage` on real data) — see Retro/FIX-001.md.

## Branch
`quenching-hogshead` (recorded in deck.md by Task 0000)

## PR table
| Repo | PR | Status |
|------|----|--------|
| cloud-manager-web | [#19](https://github.com/talkersoft-com/cloud-manager-web/pull/19) | merged |
| workflow-exec | [#67](https://github.com/talkersoft-com/workflow-exec/pull/67) | merged |

(PR-table fill ships as a follow-up PR on `blessed-countryside` — the deck
auto-merges and auto-transitions on ship.)

## Phase summary
| Phase | Delivered |
|-------|-----------|
| 0000 | hv_status → hv_next → branch `quenching-hogshead`; recorded in deck.md |
| 0001 | CM6 packages; `AnsibleYamlEditor` with C-ED prop surface; legacy trio (`initialContent`/`onSave`/`onDirtyChange`) behavior parity (dirty, Cmd/Ctrl-S, change summary, beforeunload, buffer-is-truth); one theme from `--color-*` tokens (zero hardcoded hex, grep-asserted); vitest harness; bundle guard wired into `npm run build` |
| 0002 | `@codemirror/lang-yaml` host language; Ansible overlay (play/task keywords in key position, module-name = first non-meta key of task mappings incl. block/rescue/always); indent guides; resilient on broken YAML |
| 0003 | In-repo Lezer grammar `jinja2.grammar` (no npm Jinja dependency) compiled by `scripts/generate-jinja2-parser.mjs` at build time; mounted into plain/quoted/block scalars via `parseMixed` overlays; distinct tags for delimiters/identifiers/filters; `{%` statements get delimiter + keyword |
| 0004 | `fragmentMode`: synthetic nesting-chain wrap (sequence chunks get a `tasks:` context) with constant-offset position mapping; raw buffer untouched → byte-identical saves; client lint (Lezer error nodes + unclosed-quote + tab-indent) merged with the `diagnostics` prop on one `setDiagnostics` surface; squiggles + gutter + lint panel |
| 0005 | `decorations` → kind-styled marks (`cm-deco-secret/required/vaultref`, data-varname attrs) and `hoverProvider` → hoverTooltip bridge resolving Jinja2 identifiers — both compartment-based, inert by default, exercisable purely from props (P8 plugs in without touching P7 files); both consumer pages swapped to the lazy `AnsibleYamlEditor`; regression suite green |
| 0006 | Deploy + smoke + results + ship |

## Deploy
- Target: `deploy-cloud-manager.py --target web` → rsync to `/opt/cloud-manager-web/` — exit 0, post-deploy poll **HTTP 200 after 2s** (attempt 1/10).
- Second deploy after FIX-001 (same target): **HTTP 200 after 2s** (attempt 1/10).
- No API/MCP/vorch deploys; no migrations (web-only plan).

## Smoke (https://ubuntu-server.talkersoft.com — deployed stack)
- `AnsiblePlaybookEditorPage` (pg-14-jammy): CM6 renders (no `<textarea>`), 6 Ansible-keyword marks in viewport, module names (`ansible.builtin.apt`, `community.postgresql.*`) styled, `{{ }}` Jinja2 delimiter tokens present.
- Theme: `[data-theme]` dark→light flips editor background `rgb(42,43,48)` → `rgb(255,255,255)` (token-driven, no editor-side switch).
- Save round-trip (p6-smoke-dirty fixture): edit → dirty dot + enabled Save; change summary "p7 smoke test"; Ctrl-S → PATCH; GET returned original + typed marker **byte-identical**; fixture restored afterwards.
- `AnsibleRoleDetailPage` (temp role `p7-smoke-role`, file `tasks/fragment.yml`, 4-space outer indentation): renders with full highlighting and **zero diagnostics** (no false "missing document start"); typing `bad: "unclosed` produced 1 error squiggle + 1 gutter marker live. Temp role deleted (HTTP 204).
- List page `/ansible/playbooks`: only `index-*.js` fetched — editor chunk not loaded.

## Bundle budget
- Recorded budget: **375,091 bytes**; deployed editor chunk `AnsibleYamlEditor-BfahXB5k.js` = **377,403 bytes** (within ±10%). Entry chunk asserted CodeMirror-free; guard runs in `npm run build` (invoked by `.cicd/build-cloud-manager.py`) and fails outside ±10% (failure path verified).

## Scope guard
Code changes in `cloud-manager-web` only (+ this orchestration folder in
`workflow-exec`). No diffs in cloud-manager-api, cloud-manager-mcp, vorch-lib,
or vorch-service.
