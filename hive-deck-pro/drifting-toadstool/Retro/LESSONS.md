# Lessons: hive-deck-pro/drifting-toadstool

1. **Golden-first pays for itself.** Capturing pre-change output with a binary built from
   HEAD before touching code caught a real regression (FIX-001: deck listing order) that
   every unit test missed. Byte-identical goldens are a much sharper back-compat bar than
   "tests pass".

2. **Sort what you printed, not what you derived.** The decks listing sorted glob paths
   (with `.yaml`); re-implementing the union over trimmed stems silently changed ordering
   for names that prefix other names (`cloud-manager` vs `cloud-manager-inactive`).
   When refactoring a listing, keep the exact sort key.

3. **Plan-decided defaults removed all ambiguity.** The plan carried three open questions
   with decided defaults (HV_PROFILE identity, claude-profiles fallback, `deck_config`
   naming). Execution never had to stop and ask.

4. **Layered lookup wants one wrapper, not edits at every call site.** Wrapping
   `findConfigFile`/`findInConfigSubdir` with layered variants kept the diff small and made
   "config.yaml never falls back" trivially auditable (it is the one caller left on the
   un-layered function).

5. **HV_PROFILE mode is a semantic opt-in.** In profile mode the machine root is the thin
   profile dir and a missing artifact is NOT an error (unlike the legacy HV_HOME contract,
   where every file must exist at the root). Keeping the two modes on separate code paths
   avoided bending the legacy error contract.

6. **The plan PR lived on the `hive` branch, not `main`.** PLAN.md wasn't on disk after
   `hv_next` (branches base on origin/main); it had to be read from the merged planning
   commit (`git show`). Worth knowing for future orchestrations on this deck:
   default_target_branch is `hive`.

7. **Background shells buffer; pkill by command-line is a footgun.** A fixture script run
   in the background appeared hung (buffered stdout), and a later `pkill -f` matching the
   hv invocation killed the foreground shell group too. Run short verification commands in
   the foreground with timeouts.
