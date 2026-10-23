# blip-cli-skill

An agent skill for sending files and folders to devices on your own [Blip](https://blip.net/) account through [SahilUX/blip-cli](https://github.com/SahilUX/blip-cli). The skill is named `blip-cli`. It guides recipient selection, waits for delivery, and checks transfers after timeouts.

This repository contains instructions only. You must install the CLI separately, with its native Blip engine, and sign in to a Blip account.

**CLI availability:** `SahilUX/blip-cli` is currently private. You need access to that repository to install the implementation this skill supports. Installing this public skill does not grant access or provide a replacement CLI. The CLI link may return 404 when you are not authorized.

## Prerequisites

Linux x86-64 with glibc has verified existing-session transfers, shutdown, and recovery with systemd. Linux aarch64 and fresh email-code login remain experimental.

macOS is a preview. Official installation, vendor signature verification, and native startup, status, and stop have been verified on Apple Silicon running macOS 27.0.1 with Blip 1.2.3. Real email-code login and authenticated transfers from the Mac CLI to Linux and the Mac desktop app have been verified. File and nested-folder transfers completed with matching SHA-256 checksums. Native stop/start/status preserved the signed-in device. Real-service logout and unattended or SSH operation on macOS remain unverified. The archive also contains Intel code, but Intel runtime support is experimental.

The sender needs:

- Node.js 22 or newer. The client has no npm dependencies.
- `git`, `curl`, and a C compiler available as `cc` for installation.
- On Linux, `tar` and `sha256sum`. On macOS, `unzip`, `shasum`, and `codesign`. If `cc` is missing on macOS, run `xcode-select --install` to install Apple's Command Line Tools.
- The separately installed [blip-cli](https://github.com/SahilUX/blip-cli#readme), with its command on `PATH`. The default command is `blip`.
- The native Blip engine installed by the CLI's installer, and a signed-in Blip account.
- Blip installed and signed in to the same account on the receiving device. Keep Blip open on phones and computers awake during a transfer.

The installer verifies a pinned SHA-256 checksum of the official archive and builds a small C host in `~/.local/lib/blip-cli`. Linux uses the Blip 1.2.6 native library. macOS uses the intact, vendor-signed `BlipKit.framework` from Blip 1.2.3, including its signature resources and `Info.plist`. The host launches no GUI executable or Java runtime. It needs no Xvfb or D-Bus session; macOS still uses Apple framework dependencies.

Follow the [CLI README](https://github.com/SahilUX/blip-cli#readme) for installation and interactive login. A skill clone installs none of these prerequisites. For commercial activities, Blip's terms require a Business plan.

### Linux sessions

Existing Linux sessions use `~/.local/share/net.blip.Blip` and the socket `~/.cache/net.blip.Blip/sock`. Stop the old runtime and update any service that launches `~/.local/bin/blip` before installing, because that path becomes the CLI. Follow the CLI's desktop migration instructions to preserve the existing session.

### macOS preview profile

The CLI uses a separate profile and requires fresh CLI login. The CLI does not read or migrate the desktop app's credential files or stop the desktop app. The native framework may use Apple system storage, so the separate profile does not guarantee that all credentials stay under the CLI data directory.

- Engine data: `~/Library/Application Support/blip-cli/engine`
- Socket: `~/Library/Caches/blip-cli/engine/sock`
- Logs, PID records, and temporary files: `~/Library/Caches/blip-cli`
- Remembered recipient: `~/Library/Application Support/blip-cli/config.json`, unless an absolute `XDG_CONFIG_HOME` selects another config directory

`BLIP_APP` defaults to `~/.local/lib/blip-cli/blip-core` on both platforms. On macOS, `BLIP_LIB` defaults to `~/.local/lib/blip-cli/BlipKit.framework/Versions/A/BlipKit`. These select the native host and library. Use the CLI installer to retain the complete signed framework.

## Install the skill manually

Install into the directory your agent scans for skills, using `blip-cli` as the folder name. Run one of these examples on the machine where the agent discovers skills. Execute transfers on the configured sender, within the verified platform limits above.

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
