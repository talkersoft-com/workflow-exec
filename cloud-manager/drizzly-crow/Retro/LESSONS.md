# Lessons: cloud-manager/drizzly-crow

## What worked

- **Symlink bridge pattern**: pointing HV_HOME at a parent dir and symlinking `.hv → repo` preserved full backward compatibility — no callers needed to change their path expectations.
- **configure script over manual steps**: wrapping env-file write + symlink creation in `make configure` means the setup is one command and self-documents in the Makefile.

## What tripped us up

- **FindHome() strips path levels**: the initial attempt pointed HV_HOME directly at the repo. `config.FindHome()` strips two path segments from `config.yaml`'s location to derive the project root, so it landed one level too high. Discovered this during end-to-end verify (Task 0004), not in unit tests — the unit tests only cover `findConfigFile`, not the full `LoadSetup()` pipeline.
- **Session ended before Task 0004**: ORCH.md checkboxes were never updated. Future sessions resuming a /loop should re-read ORCH.md state from the task files and git log rather than trusting checkboxes alone.

## Recommendations

- Add an integration test that exercises the full `LoadSetup()` with HV_HOME set to a temp dir (no `.hv/` subdirectory) to catch `FindHome()` regressions.
- Consider documenting the symlink convention (`config/.hv → workflow-configuration`) in the deck's README.
