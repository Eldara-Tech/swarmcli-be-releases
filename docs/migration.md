# Migrating to application-layer mTLS

Releases up to and including **v1.6.0** put the internal `swarmcli-agent-net`
overlay in **encrypted** mode. Docker tunnels encrypted overlays over IPsec
**ESP (IP protocol 50)**. On clusters that allow the standard overlay ports to
a worker node but block ESP, the `agent-manager → worker-agent` hop is silently
dropped, so **shells (and port-forwards) into tasks on worker nodes time out**
while manager-node tasks work.

The fix moves confidentiality and authentication off the overlay and onto
**application-layer mutual TLS** on the `rbac-proxy → agent-manager → agent`
hops, so `swarmcli-agent-net` can be a plain overlay that needs no ESP. A
Docker overlay's `encrypted` option cannot be changed in place, so moving to
the new model is a one-time migration that recreates the overlay — it is
**not** an images-only `:bootstrap --upgrade`.

## The one command

After upgrading the `swarmcli-be` binary (see [Installation →
Upgrade](installation.md#upgrade)):

```
:bootstrap --migrate
```

`:bootstrap --migrate` shows a confirmation summarising exactly what is kept
and what is recreated, then performs the migration on accept. Port and proxy
node are detected from the running stack — you don't pass any other flags.

> `:bootstrap --upgrade` (images-only) deliberately **refuses** a pre-mTLS
> stack and points you here: an images-only update cannot change the overlay.

## It is non-destructive

The migration **preserves your identity and data** — there is no
re-onboarding and no certificate redistribution:

| Preserved (untouched) | Recreated |
|---|---|
| User CA and all issued user certificates | `swarmcli-agent-net` (now a plain overlay) |
| Admin token | The internal mTLS certificates (a dedicated internal CA) |
| Your managed Docker context | The infra services (rolling redeploy) |
| The RBAC user database | |

The only interruption is the few seconds the stack takes to reconverge: active
shell sessions and port-forwards drop and must be reopened. Existing user
Docker contexts keep working afterwards — their certificates are unchanged.

## Order of operations

1. **Upgrade the binary first** so it deploys the matched, mTLS-capable agent
   and rbac-proxy images:
   ```
   brew upgrade swarmcli-be          # Homebrew
   scoop update swarmcli-be          # Scoop
   docker pull eldaratech/swarmcli-be:latest   # Docker
   ```
2. From the **original (non-managed) context** on a swarm manager, run
   `:bootstrap --migrate` and accept the confirmation.
3. Switch back to your managed context with `:contexts` if needed — it was
   preserved, so nothing else changes.

Migration is opt-in: an un-migrated stack keeps working for manager-node
shells. Worker-node shells on ESP-blocked clusters keep failing until you
migrate.

## Verifying

- `:bootstrap --check` reports the deployed component versions.
- A shell (`x`) into a task on a **worker** node now connects instead of timing
  out with `waiting for attach: … i/o timeout`.

If a worker-node shell still reports that the stack uses the *legacy encrypted
agent-net overlay*, the migration has not been run yet — run `:bootstrap
--migrate`.
