# SwarmCLI Business Edition

Docker Swarm TUI — Business Edition. A single-binary superset of
[SwarmCLI](https://github.com/Eldara-Tech/swarmcli) (CE) that adds:

- `:bootstrap` — one-command deploy of an mTLS-fronted RBAC proxy and a
  per-node agent stack onto your Swarm.
- Per-user RBAC, identity by client certificate.
- Interactive shell into a running service task (`x` on a service).
- Reveal-secret for debugging (`x` on a secret).

## Install

### Homebrew (macOS / Linux)

```bash
brew install Eldara-Tech/tap/swarmcli-be
```

### Scoop (Windows)

```powershell
scoop bucket add eldara https://github.com/Eldara-Tech/scoop-bucket
scoop install swarmcli-be
```

### Docker

```bash
docker pull eldaratech/swarmcli-be:latest
```

### Binary

Download from the [Releases](https://github.com/Eldara-Tech/swarmcli-be-releases/releases) page.

The installed executable is named `swarmcli` (BE is a strict superset of
CE — same binary name, expanded feature set). See [docs/installation.md](docs/installation.md)
for details.

## Documentation

End-user documentation lives in [`docs/`](docs/):

- [Installation](docs/installation.md)
- [License](docs/license.md)
- [Bootstrap](docs/bootstrap.md)
- [RBAC](docs/rbac.md)
- [Features](docs/features.md)
- [Configuration](docs/configuration.md)
- [Troubleshooting](docs/troubleshooting.md)

## Related

- [swarmcli](https://github.com/Eldara-Tech/swarmcli) — Community Edition (OSS, Apache 2.0).
- [swarmcli-agent](https://github.com/Eldara-Tech/swarmcli-agent) — per-node agent and agent-manager (deployed by `:bootstrap`).
- [swarmcli.io](https://swarmcli.io) — product home and license sign-up.
