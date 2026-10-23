# blip-cli-skill

An agent skill for sending files and folders to devices on your own [Blip](https://blip.net/) account through [SahilUX/blip-cli](https://github.com/SahilUX/blip-cli). The skill is named `blip-cli`. It guides recipient selection, waits for delivery, and checks transfers after timeouts.

This repository contains instructions only. You must install the CLI separately, with its native Blip engine, and sign in to a Blip account.

**CLI availability:** `SahilUX/blip-cli` is currently private. You need access to that repository to install the implementation this skill supports. Installing this public skill does not grant access or provide a replacement CLI. The CLI link may return 404 when you are not authorized.

## Prerequisites

Run the CLI on a Linux sender with glibc on x86-64. Linux aarch64 installation and fresh email-code login are experimental. Existing-session transfers, shutdown, and recovery with systemd have been tested. macOS support is not yet verified.

The sender needs:

- Node.js 22 or newer.
- `curl` and `git` for installation.
- A C compiler available as `cc`, plus `tar` and `sha256sum`, to install the native runtime.
- The separately installed [blip-cli](https://github.com/SahilUX/blip-cli#readme), with its command on `PATH`. The default command is `blip`.
- The native Blip engine installed by the CLI's installer, and a signed-in Blip account. The installer verifies the pinned SHA-256 checksum of the official Blip 1.2.6 archive, retains the native library, and builds a small C host in `~/.local/lib/blip-cli`. It does not install the GUI app or Java runtime. No Xvfb or D-Bus session is needed. The Node.js client has no npm dependencies.
- Blip installed and signed in to the same account on the receiving device. Keep Blip open on phones and computers awake during a transfer.

Follow the CLI README for installation, migration from a desktop runtime, and interactive login. Existing Linux sessions use `~/.local/share/net.blip.Blip` and the socket `~/.cache/net.blip.Blip/sock`. Stop the old runtime and update any service that launches `~/.local/bin/blip` before installing, because that path becomes the CLI. A skill clone installs none of these prerequisites. For commercial activities, Blip's terms require a Business plan.

## Install the skill manually

Install into the directory your agent scans for skills, using `blip-cli` as the folder name. Run one of these examples on the machine where the agent discovers skills. Execute transfers on the configured Linux sender.

For Codex's default user skills directory:

```sh
mkdir -p ~/.codex/skills
git clone https://github.com/SahilUX/blip-cli-skill.git ~/.codex/skills/blip-cli
```

For Claude Code's user skills directory:

```sh
mkdir -p ~/.claude/skills
git clone https://github.com/SahilUX/blip-cli-skill.git ~/.claude/skills/blip-cli
```

For another agent or a customized skills location, clone into that agent's skills directory under `blip-cli`. The installed entrypoint must be `blip-cli/SKILL.md`. If the destination already exists, inspect it before updating or replacing it. Restart or reload your agent's skill discovery as needed.

To update a clone, use its installation path:

```sh
git -C ~/.codex/skills/blip-cli pull --ff-only
```

To remove the skill, remove its clone from the agent's skills directory. This does not uninstall the CLI or Blip.

## Use the skill

Ask your agent to use `blip-cli`, for example:

> Use blip-cli to send `/absolute/path/report.pdf` to my device named "Target device".

The agent checks the available devices and passes `--to` explicitly. It reports delivery only after `send` exits successfully. If a wait times out, the agent checks the existing transfer before considering a resend. The CLI supports devices on your own account, not sending to other people's accounts.

See [SKILL.md](SKILL.md) for the workflow and the [CLI README](https://github.com/SahilUX/blip-cli#readme) for command documentation. The CLI uses undocumented app internals and can break after Blip updates.

## Rights and attribution

[Blip](https://blip.net/) is software and a service provided by Blip Studio Inc. This project is independent and unofficial. It is not affiliated with, endorsed by, or sponsored by Blip Studio Inc.

The [MIT license](LICENSE) applies to this repository's original skill and documentation text, copyright 2026 Sahil Saleem. It grants no rights to Blip's proprietary software, services, content, or trademarks. This repository includes no Blip logos, app binaries, or decompiled code. The separate CLI has its own MIT license and [legal provenance notice](https://github.com/SahilUX/blip-cli/blob/main/LEGAL.md), accessible to authorized repository users.

Blip's [terms of service](https://blip.net/terms-of-service), reviewed October 17, 2026 and marked updated January 18, 2025, reserve Blip's trademark and content rights, restrict trademark use without permission, and grant limited intended-purpose use. They require a Business plan for commercial activities.

The CLI relies on reverse-engineered, undocumented local RPC behavior. Permission for that reverse engineering, RPC use, or trademark use has not been established by this project. Attribution and the MIT license do not resolve those questions or establish legal clearance. Review Blip's current terms before use.
