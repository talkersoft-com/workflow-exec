# Wishlist: Craft Ingot

> EXAMPLE FILE — This shows the pattern for a self-improvement wishlist. Write one of these at the end of every workflow run, after the result. Do not modify this file.

## Wishlist ID
`0001-CRAFT-INGOT-WISHLIST-001`

## Source
- Orchestration: `Orchestrate/0001-CRAFT-INGOT-ORCH.md`
- Result: `Results/0001-CRAFT-INGOT-RESULT.md`

## What This Is

A reflection — *not* a complaint log. After completing the workflow, identify the tools, MCPs, docs, or access I wished I had. The intent is to give the human operator concrete feedback about where tooling investment would compound across future workflows.

Three things to look for while writing this:

1. **Guesses I had to make.** Anywhere I had to infer instead of read or query is a sign that ground truth was missing.
2. **Out-of-band verification.** Anything I had to shell out to do (curl, psql, virsh) that an MCP could expose as a tool is a candidate.
3. **Repeated friction.** If the same kind of lookup or check came up in multiple tasks, that pattern is worth tooling.

## Wishlist

### Wished I had: `furnace_status` MCP tool
**Friction**: I had to physically walk to the furnace to check whether it was occupied. An MCP method `furnace_status(furnace_id) → { occupied, contents, fuel_remaining }` would have let me decide between the primary and blast furnace before attempting to load ore, saving the failed first attempt + the improvisation cycle.
**Suggested signature**: `furnace_status(furnace_id: string) → FurnaceState`

### Wished I had: a docs page listing fallback equipment
**Friction**: I had to discover the blast furnace existed by walking around. A short doc at `docs/equipment/smelting.md` listing each smelting station with capabilities + fallback rank would have made the improvisation deterministic rather than exploratory.
**Suggested location**: `docs/equipment/smelting.md`

### Wished I had: read access to inventory baseline before/after
**Friction**: TC-002 (no ore lost) required me to remember the ore count before starting. I had no API to snapshot inventory state at task start. An `inventory_snapshot()` MCP would make pre/post diffs trivial.

## Smooth Things (worth keeping)

- The improvisation file pattern worked exactly as intended — when I hit the occupied-furnace failure, writing the adaptation file before retrying gave me a clear cognitive separation between "failure" and "fix."
- The test file's TC-001/TC-002/TC-003 breakdown caught the partial-success case where ore was consumed but no ingot produced — without that granularity I would have called it done at "ingot present."

---

> PATTERN NOTE — Be specific. "I wish I had better tooling" is useless. "I wish I had `furnace_status(id)` because I lost ~5 minutes walking to discover occupancy" is actionable. Name the MCP method or doc path you imagine. If the friction was a one-off, say so — not every wish needs to become work. If everything went smoothly, write a one-line wishlist saying so; the absence of friction is itself signal worth recording.
