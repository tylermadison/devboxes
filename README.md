# devboxes

This repo holds two launchers that run AI coding agents inside locked-down Docker containers:

- **ccbox** runs Claude Code.
- **pibox** runs the Pi coding agent (pi.dev).

Each launcher runs the agent against the project in your current directory. The container can write only to that project. It mounts read-only copies of selected host settings, and it never mounts your credentials. Network egress can be limited to an allowlist (opt-in).

## Contents

- `ccbox.zip` — the ccbox package: launcher, Dockerfile, allowlist, and startup scripts.
- `pibox.zip` — the pibox package, with the same layout.
- `ccbox README.md` and `pibox README.md` — the full documentation for each tool.

The source files are inside the zip archives. To work on them, unzip into a separate directory and do not edit them in place.

## Install

```
$ unzip ccbox.zip -d ~/.local/share
$ ln -s ~/.local/share/ccbox/ccbox ~/.local/bin/ccbox
$ ccbox build          # CLAUDE_CODE_VERSION=x.y.z ccbox build to pin
```

pibox installs the same way, with `pibox` in place of `ccbox` and `PI_VERSION` in place of `CLAUDE_CODE_VERSION`.

## Use

```
$ cd ~/code/my-app
$ ccbox                # run Claude Code in this project
$ ccbox shell          # bash inside the container
$ ccbox reset          # delete the container's home volume (logins, history, caches)
```

See the tool READMEs for authentication, environment options, and the threat model.
