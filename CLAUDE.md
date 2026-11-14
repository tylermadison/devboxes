# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Two sibling tools that run AI coding agents in locked-down Docker containers: `ccbox` (Claude Code) and `pibox` (Pi coding agent). The source lives inside `ccbox.zip` and `pibox.zip`, which are untracked. The tracked `README.md` is the only plain file. Unzip into a scratch directory to read or change the code. Do not edit files in place inside the repo.

## Commands

No build, lint, or test configuration exists. `bash -n <script>` checks syntax only.

```
unzip ccbox.zip -d <dir>             # or pibox.zip
ccbox build                          # CLAUDE_CODE_VERSION=x.y.z to pin (pibox: PI_VERSION)
ccbox [claude args]                  # run in $PWD
ccbox --yolo                         # adds --dangerously-skip-permissions
ccbox shell                          # bash in the container
ccbox reset                          # delete the ccbox-home volume
```

The launcher must run under stock macOS bash 3.2. Do not use associative arrays, `readlink -f`, or other bash 4+ features.

## Architecture

The launcher runs on the host. It builds a `docker run` command. The container runs an entrypoint that sets up the firewall and then starts the agent.

1. **Launcher** (`ccbox`, `pibox`): hardens the container with `--cap-drop ALL`, `--read-only`, `no-new-privileges`, and resource limits. It mounts `$PWD` at `/workspace` (read-write) and a named volume at `/home/node`. Host config items are mounted read-only under `/host`. Individual items are listed, not the whole `~/.claude` or `~/.pi/agent` directory, because those hold credentials. Files named `.env*`, `*.pem`, `*.key` and similar are masked with an empty read-only file. Optional features are enabled one at a time by environment variables (`CCBOX_MCP`, `CCBOX_SSH`, `CCBOX_GH`, `CCBOX_PORTS`, `CCBOX_ALLOW`, and similar).
2. **Entrypoint** (`scripts/entrypoint.sh`): runs as root. If the firewall is enabled, it runs `init-firewall.sh`. Then it drops to `node` with `setpriv` and starts `tini` and `user-init`.
3. **Firewall** (`scripts/init-firewall.sh`): iptables and ipset. It allows traffic only to domains in `allowlist.txt`, to GitHub CIDRs from `api.github.com/meta`, and to the DNS resolver. It blocks IPv6. It pins the allowed IPs in `/etc/hosts`. It fails closed.
4. **User init** (`scripts/user-init.sh`): runs as `node`. It copies host config from `/host` into the home volume. The host copy is re-synced on each start. The container cannot write back to the host.
5. **Image** (`Dockerfile`): Node base image, the agent installed globally (`@anthropic-ai/claude-code` or `@earendil-works/pi-coding-agent`), and the container user's UID and GID set to match the host.

## Things to know

- **Firewall default is open.** The launcher passes `CCBOX_FIREWALL=${CCBOX_FIREWALL:-off}` (pibox: `PIBOX_FIREWALL`). The egress allowlist is therefore not enforced unless the user sets the variable to `on`. The `ccbox README.md` text describes default-deny egress. Treat that claim as unverified until the default changes.
- `ccbox` and `pibox` are near-copies. `entrypoint.sh` is identical apart from names. Changes to the firewall, user-init, or launcher usually need the same change in both. Keep them in sync.
- The Dockerfile pins the Node base image (ccbox: `node:22`, pibox: `node:24`). Those versions differ on purpose or by accident. Check before aligning them.
