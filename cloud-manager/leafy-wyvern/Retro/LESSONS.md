# cloud-manager leafy-wyvern — Lessons

## Non-obvious things worth remembering

### 1. The consumer's `destroy_requested` write was a duplicate, not the source of truth

The audit timeline already had a `destroy_requested` event being written by the
consumer when the vorch ack came back. Moving the write into
`OrchestrationController.DestroyVirtualMachine` and *removing* the consumer
call was the right fix — the consumer's job is to record outcomes
(TeardownSucceeded / TeardownFailed), not user intent. Leaving both writers in
place would have produced duplicate timeline rows. Always grep for the existing
writer before adding a new one for the same event type.

### 2. One file per tier kept the tool-authoring work conflict-free

Tier 2 (playbook internals, 21 tools), Tier 3 (collections, 13), and Tier 4
(feature flags, 3) were each authored in a brand-new file
(`playbook-tree.ts`, `collections.ts`, `feature-flags.ts`) with only a small
edit to `index.ts` to register them. Result: zero merge conflicts even with
parallel work, and the diff per file is reviewable in isolation. **Pattern to
repeat for future tool batches:** new tier → new file; only touch `index.ts`
and `profiles.json` to wire them up.

### 3. `profiles.json` is easy to forget

The `hv.cloud` profile in `profiles.json` gates which tool categories the MCP
actually exposes. New tool files won't show up to the operator until they're
listed there — the build will be clean and CI will pass regardless. Add the
category name to the profile in the same PR as the new tool file, or operators
have to wait for a follow-up reload.

### 4. MCP reload is operator-only

The CLI cannot reload its own MCP server connection — only the operator can via
`/mcp` in Claude Code. Every PR that adds tools must surface that reminder
prominently in the operator action callout; otherwise the tools sit in `dist/`
unreachable.

### 5. Commit message wording vs. mental model

The commit message `vm-events: write destroy_requested at DELETE time, not at
ack time` is the user-facing framing; mechanically the change is "move the
`DeleteVirtualMachineAsync` call from `ConsumerService` to
`OrchestrationController` and let the service write its own audit event."
Keep both framings (intent + mechanics) in the RESULT doc so future readers
can grep for either.
