# Shared config layer + canonical path tokens (drifting-toadstool)

## What this workflow does
Implements R7.3–R7.6 in hive-deck: machine config.yaml declares a `deck_config:` shared layer;
every artifact (deck yamls, workflow extensions, fragments, modules, claude-profiles,
gitignore-rulesets) resolves machine-first then layer per FILE with union scans for listings and
the workflow registry; config.yaml never falls back. Canonical `{{WorkspaceRoot}}` token (legacy
`{{DecksRoot}}` alias kept), `~` expansion, relative-to-declaring-file paths — in config keys and
mcps.yaml registries.

## Read before starting
- `deck.md` — scope, orchestration id, hv MCP calls
- `Execution/Exec.md` — task list and execution loop
- `../../../workflow-plans/hive-deck-pro/drifting-toadstool/PLAN.md` — before/after contracts,
  resolution rules; open-question DEFAULTS ARE DECIDED: (1) HV_PROFILE-based machine identity
  with root-config fallback, (2) claude-profiles.yaml gets per-file fallback like the rest,
  (3) key is named `deck_config`.

## Constraints
- **Back-compat is the acceptance bar**: no `deck_config` → byte-identical behavior, proven by a
  golden comparison of assembled plan+workflow output before/after the change.
- Per-FILE fallback only — never key-level yaml merging across layers.
- `config.yaml` is machine identity — machine root only, no fallback, ever.
- Code changes in the hive-deck-pro repo ONLY (workflow-configuration migration is a follow-up).
- hive-deck internal rules: rebuild Go binary + mcp dist after changes; tests proportional;
  install the new binary only AFTER its tests pass (the run itself uses hv via the MCP).
