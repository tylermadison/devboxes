**cc*box*** [Install](#install)[Auth](#auth)[Usage](#usage)[Options](#options)[Threat model](#model)

Claude Code box · README

# Your Claude setup, in a *sealed* box.

ccbox runs Claude Code against any project in a locked-down container. It brings your `CLAUDE.md`, settings, agents, commands, skills and git identity, but it can only write to the project you launched it from and can only reach the network hosts you allow.

default-deny egressread-only root fsno capabilitiesno sudohost config copied, never shared rw

your machine

- `~/.claude/CLAUDE.md`ro
- `settings.json`ro
- `agents · commands · skills`ro
- `~/.gitconfig`ro
- `$PWD`rw
- `~/.ssh · credentials`never

ccbox · node

- `/host/* → ~/.claude`copied
- `/workspace`rw
- `/home/node`volume
- `/ rootfs`read-only
- `caps · sudo`none
- `$ claude --dangerously-skip-…`

egress gate

- `api.anthropic.com`allow
- `registry.npmjs.org`allow
- `github CIDRs`allow
- `CCBOX_ALLOW=…`allow
- `everything else`reject
- `IPv6`reject

Install

## Three commands to *build* it.

Unzip or clone into `~/.local/share/ccbox`, put the launcher on your path, then build the image. The build matches the container user to your host UID/GID, so files in your project stay yours.

zsh

```
$ ln -s ~/.local/share/ccbox/ccbox ~/.local/bin/ccbox
$ chmod +x ~/.local/share/ccbox/ccbox
$ ccbox build          # CLAUDE_CODE_VERSION=x.y.z ccbox build to pin
```

Auth

## Your host credentials are *never* mounted.

Choose one of these three ways to sign in.

Login once

### Inside the box

Run `ccbox`, type `/login`, then open the URL on your host. The login is saved in the `ccbox-home` volume.

Keychain

### Long-lived token (macOS)

Run `claude setup-token` on the host, then `security add-generic-password -s ccbox -a "$USER" -w '<token>'`. ccbox reads the token and passes it in as `CLAUDE_CODE_OAUTH_TOKEN`.

API key

### Environment

If `ANTHROPIC_API_KEY` is set on the host, ccbox passes it in as is.

Usage

## Run it from *any* project.

\~/code/my-app

```
$ ccbox                     # claude
$ ccbox --yolo              # claude --dangerously-skip-permissions — what the box is for
$ ccbox -p 'explain src/'   # args pass straight through to claude
$ ccbox shell               # bash in the box
$ CCBOX_PORTS=3000 ccbox    # next dev on 127.0.0.1:3000 (run it with -H 0.0.0.0)
$ ccbox reset               # wipe the home volume: history, login, caches
```

Options

## Opt-ins widen the box, *one* at a time.

Every option is an environment variable. To replace the built-in allowlist permanently, put your own at `~/.config/ccbox/allowlist.txt`.

| Variable | Default | Effect |
| --- | --- | --- |

Threat model

## What it isolates, and what it *doesn't*.

Isolated

Not isolated

ccbox · Dockerfile, entrypoint, egress firewall and launcher. Works with the stock bash 3.2 on macOS.