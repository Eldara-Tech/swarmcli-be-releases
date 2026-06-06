# Troubleshooting

Common errors and fixes for Business Edition deployments. For issues with
general TUI behaviour, see the
[swarmcli](https://github.com/Eldara-Tech/swarmcli) issue tracker.

## License

**"Your license key is invalid."** at startup.
The key did not pass signature verification. Possible causes: the key was
truncated when copied, an extra newline was included, the key was issued
against a different public key, or the schema version is below
`MinSchemaVersion`. Re-paste the key from its source. If it persists,
contact whoever issued the key.

**"Your license expired on YYYY-MM-DD. The 5-day grace period has ended."**
The key is past its grace window. Paste a fresh key at the prompt, or set
`SWARMCLI_LICENSE` and restart. See [License — Lifecycle](license.md#lifecycle-states).

**`:license` shows "Node limit exceeded" / "User limit exceeded".**
Soft limit warning; nothing is blocked. Either upgrade the license, scale
the cluster down, or treat it as informational. See
[License — Limits](license.md#limits).

## Bootstrap

**"Cannot bootstrap from a managed context (foo-managed)."**
The current Docker context is itself a managed context produced by a
previous bootstrap. `:bootstrap` refuses to operate through the rbac-proxy.
`:contexts` and select the original (non-`-managed`) context, then retry.

**"Infrastructure stack swarmcli-infra is already running."**
Detection found all three services already active. If this is intentional
and you only wanted a status report, run `:bootstrap --check`. If you
genuinely want to redeploy, follow
[Bootstrap — Re-bootstrap](bootstrap.md#re-bootstrap-and-teardown) — do
not reach for `--force` on a live cluster.

**A worker-node shell times out (`waiting for attach: … i/o timeout`) while
manager-node shells work**, or `:bootstrap --upgrade` reports the stack
*"predates application-layer agent-net TLS (encrypted overlay)"*.
The stack still uses the legacy encrypted `swarmcli-agent-net` overlay, whose
lower-level transport some clusters block, dropping connectivity to worker
nodes. Run `:bootstrap --migrate` to move to application-layer mTLS. It is
**non-destructive** — your CA, admin token, managed context and RBAC database
are preserved (no re-onboarding). See [Migration](migration.md).

**`:bootstrap --migrate` stops with "connected through the rbac-proxy … refuses
to remove its own protected stack."**
The active Docker context reaches the daemon *through* the proxy — a managed
connection — so migrate cannot tear down the stack it would recreate. This
happens when the context is a `<name>-managed` one, or when `DOCKER_HOST` /
`DOCKER_CONTEXT` points at the proxy port. Migrate must run with **direct daemon
access**: `:contexts` and select the original (non-`-managed`) context (or unset
`DOCKER_HOST`), then retry. The stopped run makes no changes, so it is safe to
re-run once you have switched.

**Bootstrap fails at "deploy stack" with a port-in-use error.**
Another process on the manager is bound to the chosen port (default
`2376`). Either free the port, or pick another:
`:bootstrap --port 12376`.

**Bootstrap succeeds but the new managed context cannot connect** —
`Cannot connect to the Docker daemon at tcp://…:2376`.
The auto-detected proxy host is not reachable from your workstation. The
common reason is that the manager's `Status.Addr` is an internal cluster
IP (multi-NIC, NAT, or a containerised dev environment). Tear the bootstrap
down (the four commands in
[Bootstrap — Re-bootstrap](bootstrap.md#re-bootstrap-and-teardown)) and
re-run with `:bootstrap --host <reachable-host>` to bake the right value
into the cert SAN and the context endpoint.

**`:bootstrap --check` reports `agent-manager: false` but `agent: true`.**
The agent-manager service is not running on a manager. Inspect with
`docker service ps swarmcli-infra_agent-manager` and look at the latest
task's error column. Most often it's a placement-constraint miss (the
service requires a manager node and your cluster has none in the
expected state).

## Shell

**`403 exec on protected stack requires admin role` when pressing `x` on a
service.**
The current managed context belongs to a non-admin user. Switch to an
admin context with `:contexts`, or escalate the user with
`swcproxy user delete` + re-add as `--admin` (see
[RBAC — `swcproxy` CLI](rbac.md#the-swcproxy-cli)).

**`EXEC_ERROR: no shell available`.**
The container image has no `bash`, `sh`, or `ash` at the standard paths.
Set `SWARMCLI_SHELL_CMD` to a command that exists in the image (for
distroless or scratch-based images, you may have no usable shell at all
and need a debug sidecar instead).

**WebSocket connect fails immediately, no error message.**
The rbac-proxy URL is wrong or unreachable. Try
`docker --context <name>-managed ps` from a shell — if that fails, the
issue is at the Docker context / network level, not BE-specific. See
the bootstrap-host troubleshooting above.

## Reveal-secret

**"Reveal timed out — no output captured."**
The temporary service did not produce log output within 20 seconds. Most
common causes:

- The reveal image (default `alpine:latest`) is not on the node and the
  pull is slow or blocked. Use `SWARMCLI_REVEAL_IMAGE` with an image
  already cached on the node, or pre-pull.
- The node has no scheduling capacity. Inspect with
  `docker service ps swarmcli-reveal-<name>-<ts>`.
- The secret is not mounted at `/run/secrets/<name>`. This shouldn't
  happen for stock Docker, but if you see it, file an issue with the
  secret's metadata.

**Reveal returns garbled binary output.**
The secret is binary, not text. The view shows the raw bytes; if it
parses cleanly as base64 it also shows a decoded form. Use the raw value
for binary secrets.

**`Forbidden` (403) when invoking reveal.**
The current user is not an admin (reveal-secret creates a temporary
service, which is a protected mutation). Switch to an admin context.

## RBAC users

**Onboarding URL returns "token expired" or "not found".**
Onboarding tokens are single-use and time-limited (default 24 h). Generate
a fresh one with `swcproxy user regenerate-token <name>`.

**`docker context import` succeeds but `docker --context <name>-managed ps`
returns `tls: bad certificate`.**
The cert in the bundle was signed by the bootstrap CA, but the client is
validating against a different CA — usually because a `--force`
re-bootstrap rotated the CA between issuance and use. Re-onboard the
user to get a cert signed by the new CA.

**A deleted user can still connect for a while.**
This shouldn't happen — the rbac-proxy looks up the cert CN on every
request and a deleted user's record is gone immediately. If you see
otherwise, restart the rbac-proxy service to flush any caches:
`docker service update --force swarmcli-infra_rbac-proxy`.

## Managed context disappeared

**`docker context use foo-managed` says the context does not exist.**
Docker contexts live under `~/.docker/contexts/`; deletion or a clean of
that directory removes them. Re-create by re-running `:bootstrap` (which
will refuse if the stack is already running — see the teardown commands
in [Bootstrap — Re-bootstrap](bootstrap.md#re-bootstrap-and-teardown)) or
by manually re-importing if you have the original cert files saved.

## Installation

**`brew install swarmcli-be` says it conflicts with `swarmcli`.**
By design — both formulae install the same `swarmcli` binary on disk.
Brew offers to uninstall the existing one; accept, then re-run.

**`scoop install swarmcli-be` cannot find the package.**
Add the bucket first: `scoop bucket add eldara https://github.com/Eldara-Tech/scoop-bucket`.

**Docker container starts but immediately exits.**
The TUI requires a TTY. Pass `-it` to `docker run` and ensure your
terminal is interactive.

## Asking for help

If none of the above matches, capture:

1. The startup line (`swarmcli 2>&1 | head -n 1`) — confirms version and
   edition.
2. `:bootstrap --check` output (if the issue is bootstrap- or
   proxy-related).
3. The relevant log file:
   - Production builds: `~/.local/state/swarmcli/app.log` (JSON).
   - Dev builds: `~/.local/state/swarmcli/app-debug.log` (pretty).

…and open a [release-tracker issue](https://github.com/Eldara-Tech/swarmcli-be-releases/issues)
or contact the support channel that came with your license.
