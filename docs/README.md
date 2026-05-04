# SwarmCLI Business Edition — Documentation

SwarmCLI Business Edition (BE) is a single-binary superset of the open-source
[SwarmCLI](https://github.com/Eldara-Tech/swarmcli) Docker Swarm TUI. On top
of the OSS feature set, BE adds:

- **`:bootstrap`** — one-command deploy of an mTLS-fronted RBAC proxy and a
  per-node agent stack onto your existing Swarm.
- **Per-user RBAC** — identity by client certificate, role-based gating of
  destructive and exec endpoints.
- **Interactive shell into running services** (`x` on a service).
- **Reveal-secret** for debugging (`x` on a secret).

These docs cover BE-specific topics. For general TUI navigation, key
bindings, search, contexts, and OSS environment variables, see the
[swarmcli README](https://github.com/Eldara-Tech/swarmcli#readme).

## Contents

- [Installation](installation.md) — install channels, first run, edition check.
- [License](license.md) — acquiring a key, activation paths, grace period, tiers.
- [Bootstrap](bootstrap.md) — `:bootstrap` end-to-end: what gets deployed and how.
- [RBAC](rbac.md) — managing users, roles, onboarding, and revocation.
- [Features](features.md) — shell and reveal-secret in detail.
- [Configuration](configuration.md) — BE environment variables and on-disk paths.
- [Troubleshooting](troubleshooting.md) — common errors and fixes.

For install commands and the relationship between the `swarmcli-be` and
`swarmcli` packages, see [Installation](installation.md) — or the
[repository README](../README.md) for a quickstart.

## Related

- [swarmcli](https://github.com/Eldara-Tech/swarmcli) — Community Edition (OSS, Apache 2.0).
- [swarmcli-agent](https://github.com/Eldara-Tech/swarmcli-agent) — per-node agent and agent-manager (deployed by `:bootstrap`).
- [swarmcli.io](https://swarmcli.io) — product home and license sign-up.
