# Installation

SwarmCLI Business Edition ships as a single static Go binary. The executable
on disk is named **`swarmcli`** — not `swarmcli-be` — because BE is a strict
superset of CE: the OSS feature set is included unchanged. Aliases, scripts,
and CI invocations from a CE installation continue to work without
modification when you switch to BE.

The package/formula/image names *are* `swarmcli-be`, so the package manager
can offer both editions side by side. They install the same binary name, so
**you can have either CE or BE at any time, but not both**. See
[CE vs BE](#ce-vs-be) below.

## Channels

### Homebrew (macOS / Linux)

```bash
brew install Eldara-Tech/tap/swarmcli-be
```

The formula is hosted at
[Eldara-Tech/homebrew-tap](https://github.com/Eldara-Tech/homebrew-tap).
It conflicts with the CE `swarmcli` formula; brew will refuse to install
both at once and will offer to remove the other one.

### Scoop (Windows)

```powershell
scoop bucket add eldara https://github.com/Eldara-Tech/scoop-bucket
scoop install swarmcli-be
```

### Docker

```bash
docker pull eldaratech/swarmcli-be:latest
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME/.docker/contexts:/root/.docker/contexts" \
  -v "$HOME/.config/swarmcli:/root/.config/swarmcli" \
  eldaratech/swarmcli-be:latest
```

The container runs as root (matching the upstream `docker:N-cli` base, so
the mounted Docker socket works without `--group-add`). Mounting
`~/.docker/contexts` and `~/.config/swarmcli` keeps your Docker contexts
and license file persistent across container runs.

Available tags: `latest`, `vX.Y.Z`. Multi-arch images cover `linux/amd64`
and `linux/arm64`.

### Raw binary

Download platform archives from the
[Releases page](https://github.com/Eldara-Tech/swarmcli-be-releases/releases).
Each release ships:

- `swarmcli-be_<version>_<os>_<arch>.tar.gz` (Linux/macOS/FreeBSD) or `.zip` (Windows)
- `checksums.txt` — SHA-256 of every archive

Verify and install:

```bash
sha256sum -c checksums.txt --ignore-missing
tar -xzf swarmcli-be_<version>_linux_amd64.tar.gz
install -m 0755 swarmcli /usr/local/bin/swarmcli
```

Build matrix per release:

| OS | Architectures |
|---|---|
| Linux | amd64, arm64, arm (v6, v7), 386 |
| macOS | universal (amd64 + arm64) |
| Windows | amd64, arm64, 386 |

## First run

```bash
swarmcli
```

If no license is present, you'll see a startup prompt asking for a license
key (or to continue in Community Edition mode). The detail of every state
the prompt can show is in [License — Activation](license.md#activation).
Once you have a valid license, every subsequent run starts silently.

If your terminal does not look right (colors, key handling), check that
your `TERM` is set to a 256-color value — SwarmCLI expects `xterm-256color`
or equivalent.

## Verifying version and edition

There is no `--version` flag today. Two ways to check:

- **Inside the TUI**: the version is shown in the header bar; `:license`
  additionally shows license status (see [License](license.md)).
- **In the log file**: every run writes one startup line. After running
  the binary at least once, look for the most recent entry:

```bash
grep -o 'swarmcli[^"]*version=[^ "]*' ~/.local/state/swarmcli/app.log \
  | tail -n 1
# → swarmcli-be version=v1.4.0       ← BE binary
# → swarmcli version=v1.4.0          ← CE binary (also has edition=ce)
```

The log-line prefix (`swarmcli-be ` vs `swarmcli `) is the canonical
runtime signal for which edition is installed.

## CE vs BE

| | CE (`swarmcli`) | BE (`swarmcli-be`) |
|---|---|---|
| License | Apache 2.0, free | Commercial license required |
| Binary on disk | `swarmcli` | `swarmcli` (same name) |
| Distribution | Homebrew/Scoop/Docker as `swarmcli` | Homebrew/Scoop/Docker as `swarmcli-be` |
| Source repo | [Eldara-Tech/swarmcli](https://github.com/Eldara-Tech/swarmcli) | private; releases at [Eldara-Tech/swarmcli-be-releases](https://github.com/Eldara-Tech/swarmcli-be-releases) |
| Pro features | none | shell, reveal-secret, `:bootstrap`, RBAC |

Switching from CE to BE: install `swarmcli-be` (your package manager will
remove the CE `swarmcli` first), then run `swarmcli`. Existing Docker
contexts, configurations, and shell aliases continue to work; only the
binary on disk has changed.

## Upgrade

```bash
brew upgrade swarmcli-be          # Homebrew
scoop update swarmcli-be          # Scoop
docker pull eldaratech/swarmcli-be:latest   # Docker
```

For binary installs, replace the file on disk with the new archive's
contents.

The compatibility matrix between BE, the bundled CE codebase, and the
agent/rbac-proxy versions is published in each release's notes — see
[Releases](https://github.com/Eldara-Tech/swarmcli-be-releases/releases).

Upgrading the binary does not change an already-deployed infra stack. To
refresh the agent/rbac-proxy images in place, run `:bootstrap --upgrade`.
A stack first deployed before application-layer mTLS (encrypted `agent-net`)
needs a one-time, non-destructive `:bootstrap --migrate` instead — see
[Migration](migration.md).
