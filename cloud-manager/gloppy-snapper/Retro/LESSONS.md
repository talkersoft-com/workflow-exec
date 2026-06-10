# Lessons — cloud-manager/gloppy-snapper

1. **"Stale server" is a hypothesis, not a diagnosis — falsify it cheaply.** The blocker sat for a
   full session waiting on an operator `/mcp` reload that would not have fixed anything. Spawning a
   fresh server from the rebuilt dist and driving it over stdio JSON-RPC took two minutes and
   immediately disproved the theory. When a fix "needs a restart to take effect", prove the
   restarted artifact works *before* asking the operator for one.

2. **Wire integrations at the choke point the callers actually use.** The toolkit had two loader
   stacks reading two different config files (`.eng-data-tools` vs `~/.hv/toolkit/db`); the resolver
   was first wired into the one the MCP tools don't import. Before integrating, trace one real call
   (tool → import chain → file read) instead of grepping for a plausible-looking function name.

3. **Check for an existing mechanism before extending a parallel one.** A complete
   provider design (`credentialProvider`/`tunnelProvider` + `VaultCredentialProvider` +
   `SshTunnelProvider`) already existed and explains the live config's entry shape — it is dormant
   only because `factory.ts` checks the legacy field. Two vault mechanisms now coexist; the
   follow-up should converge them rather than letting a third appear.

4. **Test fixtures must match the conventions of working neighbors.** The pg-test entry lacked
   `usernameOverride` (the tester's admin-connection convention every working entry had) and its
   tunnel squatted on clouddb's port. Copying the shape of an adjacent *working* entry first would
   have avoided both failures.

5. **Plan acceptance criteria beat task-file narrowing.** Task 0001/TC-003 said "only
   database-config-loader.ts changes"; the plan said "all config load paths used by the MCP tools".
   When they conflicted, the plan was right. Tests derived from an implementation guess should be
   amended (with a FIX note) rather than obeyed.
