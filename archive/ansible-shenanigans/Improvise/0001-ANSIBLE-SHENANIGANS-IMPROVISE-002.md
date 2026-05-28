# Improvisation: Phase 0001 — No valid Vault token (blocker)

## Improvisation ID
`0001-ANSIBLE-SHENANIGANS-IMPROVISE-002`

## Source
- Task: `Tasks/0001-ANSIBLE-SHENANIGANS-TASK.md`
- Failed Step: Step 5 (`./ansible-cli configure`) — pre-flight token check

## What Failed

`configure` needs `VAULT_TOKEN` with capabilities `sudo` on `sys/policies/acl/*` and write on `auth/token/roles/*`. Two attempted sources:

1. **`~/ansible-token.txt`** — file no longer exists. It was present earlier in the prior session (`-rw------- 1 todd todd 96 May 23 19:49`) but has been removed or never recreated after the most recent restart.
2. **`$VAULT_TOKEN` from `~/.profile`** — present, 95 chars (`hvs.CAESIE52...`), but `vault token lookup-self` returns `403 permission denied`. Either the token was revoked, the TTL expired, or it was never high-privilege enough for `configure`.

## Root Cause

Out-of-band human action: the install-time token must be provisioned by an operator. The workflow has no way to mint a `sys/policies/acl/*`-capable token on its own — that's the very chicken-and-egg situation the install token is designed to solve.

## Adaptation

**None possible from here.** Phase 0001 is blocked until a valid token is provided. Pausing the workflow. The orchestration's task list checkbox for 0001 remains unchecked.

When unblocked, the operator should either:

```sh
# Option A: drop a token at the conventional path
echo "<new-token>" > ~/ansible-token.txt && chmod 600 ~/ansible-token.txt

# Option B: export it for the current shell
export VAULT_TOKEN=<new-token>
```

Then resume by re-invoking the /loop prompt from `init.md`.

## Outcome

Blocked. Awaiting human action.

## Reusable Knowledge

The `configure` step is the only operation in this whole workflow that needs a *human-provisioned* Vault token — every later phase uses the orchestrator's long-lived `cloudmanager` token (or the short-lived child tokens minted from it). When provisioning new install environments, plan for the operator to have a sudo-capable token ready at the same moment they kick off `/loop`. A future enhancement: have `init.md` enforce this as a precondition check rather than letting `configure` discover it.
