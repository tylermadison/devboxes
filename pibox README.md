**pi*box*** [Install](#install)[Auth](#auth)[Usage](#usage)[Options](#options)[vs ccbox](#diff)[Threat model](#model)

Pi coding agent box · README

# An agent that never asks, in a box that *doesn't let it out*.

pibox runs the [Pi coding agent](https://pi.dev) against any project in a locked-down container. It brings your own Pi setup (`settings.json`, `models.json`, `AGENTS.md`, extensions, skills, prompts, themes) and git identity. Pi runs tool calls without asking, and its project-trust prompt doesn't limit what tools can reach, so the box sets the limits instead.

default-deny egressmulti-provider allowlistno telemetryread-only root fsno capabilitiesauth.json never shared

your machine

- `~/.pi/agent/settings.json`ro
- `models.json · AGENTS.md`ro
- `extensions · skills · prompts`ro
- `Keychain: pibox / ccbox`keys
- `$PWD`rw
- `auth.json · trust.json · sessions`never

pibox · node

- `/host/pi → ~/.pi/agent`copied
- `/workspace`rw
- `/home/node`volume
- `PI_TELEMETRY · version check`off
- `caps · sudo`none
- `$ pi`

egress gate

- `anthropic · openai`allow
- `gemini · openrouter`allow
- `npm · github`allow
- `groq · xai · mistral …`opt-in
- `pi.dev`reject
- `everything else`reject

Install

## Three commands to *build* it.

Unzip into `~/.local/share/pibox`, link the launcher, then build. Pi is installed with `--ignore-scripts`, as its containerization guide recommends, and the build matches the container user to your host UID/GID.

zsh

```
$ unzip pibox.zip -d ~/.local/share
$ ln -s ~/.local/share/pibox/pibox ~/.local/bin/pibox
$ pibox build          # PI_VERSION=x.y.z pibox build to pin
```

Auth

## Keys go in one at a time, *only if set*.

The launcher checks about twenty provider variables and passes in only the ones it finds. Your host `auth.json` is never mounted.

Keychain

### API keys (macOS)

Use service `pibox` with the variable name as the account: `security add-generic-password -s pibox -a OPENAI_API_KEY -w '<key>'`

Environment

### Shell variables

Any supported key already exported in your shell, such as `ANTHROPIC_API_KEY`, `GEMINI_API_KEY` or `OPENROUTER_API_KEY`, is passed straight through.

Claude Pro/Max

### Reuses ccbox

If ccbox's Keychain token from `claude setup-token` exists, pibox passes it in as `ANTHROPIC_OAUTH_TOKEN`. Nothing else to do.

/login

### Inside the box

Uncomment the provider's login domains first. The OAuth redirect can't reach the container, so paste the final URL or code back into Pi. The login is saved in the `pibox-home` volume.

Usage

## Run it from *any* project.

\~/code/my-app

```
$ pibox                      # pi
$ pibox -p 'explain src/'    # args pass straight through to pi
$ pibox --model <id>         # any pi flag works
$ pibox shell                # bash in the box
$ PIBOX_PORTS=3000 pibox     # next dev on 127.0.0.1:3000 (run it with -H 0.0.0.0)
$ pibox reset                # wipe the volume: logins, sessions, installed packages
```

`pi install npm:…` and `git:…` work inside the box, and packages persist in the volume.

Options

## Opt-ins widen the box, *one* at a time.

Every option is an environment variable. Put a permanent allowlist at `~/.config/pibox/allowlist.txt`.

| Variable | Default | Effect |
| --- | --- | --- |

vs ccbox

## Same box, *different* agent.

Threat model

## What it isolates, and what it *doesn't*.

Isolated

Not isolated

pibox · Dockerfile, entrypoint, outbound firewall and launcher. Works with the stock bash 3.2 on macOS. See [Run Pi Safely](https://docs.pi.dev/latest/security) and [Containerization](https://docs.pi.dev/latest/containerization).