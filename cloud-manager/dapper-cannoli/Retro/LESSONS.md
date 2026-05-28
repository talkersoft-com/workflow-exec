# Lessons — dapper-cannoli

## make install-config does not update config.yaml
`make install-config` copies files from `.hv/` preserving their names. `config.yaml.example` → `~/.hv/config.yaml.example`. The user's `~/.hv/config.yaml` is a separate per-machine file that is never overwritten by `make install-config`. To propagate a change from `config.yaml.example` to a live install, edit `~/.hv/config.yaml` directly.

## MaybeWrite is additive — it never removes entries
`internal/claude/claude.go` `MaybeWrite` only adds missing allow entries; it does not remove entries that are no longer in the config. Removing a permission from `config.yaml` does not remove it from existing `settings.local.json` files. The fix requires either deleting the files and re-provisioning, or editing them directly (the script approach used here).

## settings.local.json can be tracked by git
If `.claude/settings.local.json` was committed before `.claude/` was added to `.gitignore`, it remains tracked. Changes to it show as DIRTY even with a gitignore entry. `git add -f` is required to stage the update. Consider running `git rm --cached .claude/settings.local.json` on repos where this file should not be tracked.

## .md file disappearance in execution-workflow
The execution-workflow repo has a recurring issue where tracked `.md` files disappear after branch switches. Restore with `git checkout HEAD -- .`. Root cause is not yet confirmed but may be related to `hv_next` checkout behavior interacting with the gitignore ruleset.
