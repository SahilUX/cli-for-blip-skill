# Agent skill for CLI for Blip

An unofficial agent skill that works with Blip through CLI for Blip. This project is independent. It is not affiliated with, endorsed by, or sponsored by Blip Studio Inc.

I made this for free, for Blip to implement if they want to. Blip has not approved or endorsed it.

The skill covers sending files and folders to devices on your own [Blip](https://blip.net/) account through [SahilUX/cli-for-blip](https://github.com/SahilUX/cli-for-blip). The skill is named `cli-for-blip`. It guides recipient selection, waits for delivery, and checks transfers after timeouts.

This repository contains instructions only. You must install the CLI separately, with its native Blip engine, and sign in to a Blip account.

## Prerequisites

Linux x86-64 with glibc has verified existing-session transfers and shutdown. Recovery was also verified under a systemd user service, a setup the CLI README does not document; it describes only cron startup. Linux aarch64, fresh email-code login, and real-service logout remain experimental.

macOS is a preview. Official installation, vendor signature verification, and native startup, status, and stop have been verified on Apple Silicon running macOS 27.0.1 with Blip 1.2.3. Real email-code login and authenticated transfers from the Mac CLI to Linux and the Mac desktop app have been verified. File and nested-folder transfers completed with matching SHA-256 checksums. Native stop/start/status preserved the signed-in device. Real-service logout and unattended or SSH operation on macOS remain unverified. The archive also contains Intel code, but Intel runtime support is experimental.

The sender needs:

- Node.js 22 or newer. The client has no npm dependencies.
- `git`, `curl`, and a C compiler available as `cc` for installation.
- On Linux, `tar` and `sha256sum`. On macOS, `unzip`, `shasum`, and `codesign`. If `cc` is missing on macOS, run `xcode-select --install` to install Apple's Command Line Tools.
- The separately installed [CLI for Blip](https://github.com/SahilUX/cli-for-blip#readme), with its command on `PATH`. The default command is `blip`.
- The native Blip engine installed by the CLI's installer, and a signed-in Blip account.
- Blip installed and signed in to the same account on the receiving device. Keep Blip open on phones and computers awake during a transfer.

The installer verifies a pinned SHA-256 checksum of the official archive and builds a small C host in `~/.local/lib/blip-cli`. Linux uses the Blip 1.2.6 native library. macOS uses the intact, vendor-signed `BlipKit.framework` from Blip 1.2.3, including its signature resources and `Info.plist`. The host launches no GUI executable or Java runtime. It needs no Xvfb or D-Bus session; macOS still uses Apple framework dependencies.

When the CLI starts the engine, and again during `blip login`, it sends the engine a user agent with the OS name and kernel release, the client name `blip-cli`, and the app ID `io.github.sahilux.blip-cli`. The engine appears to pass this on to Blip's servers, so Blip can tell that the device uses an unofficial client.

The engine keeps running after the command that started it exits. While it runs, the sender is an online Blip device that can also receive files from your other devices, without a prompt if Blip's auto-accept setting allows it. The engine also keeps the environment of the command that started it, including any API tokens. If you only want to send, run `blip stop` once your transfers finish. It stops only an engine that `blip` started.

Follow the [CLI README](https://github.com/SahilUX/cli-for-blip#readme) for installation and interactive login. A skill clone installs none of these prerequisites. For commercial activities, Blip's terms require a Business plan.

### Linux sessions

Existing Linux sessions use `~/.local/share/net.blip.Blip` and the socket `~/.cache/net.blip.Blip/sock`. Stop the old runtime and update any service that launches `~/.local/bin/blip` before installing, because that path becomes the CLI. Follow the CLI's desktop migration instructions to preserve the existing session.

### macOS preview profile

The CLI uses a separate profile and requires fresh CLI login. The CLI does not read or migrate the desktop app's credential files or stop the desktop app. The native framework may use Apple system storage, so the separate profile does not guarantee that all credentials stay under the CLI data directory.

- Engine data: `~/Library/Application Support/blip-cli/engine`
- Socket: `~/Library/Caches/blip-cli/engine/sock`
- Logs, PID records, and temporary files: `~/Library/Caches/blip-cli`
- Remembered recipient: `~/Library/Application Support/blip-cli/config.json`, unless an absolute `XDG_CONFIG_HOME` selects another config directory

`BLIP_APP` defaults to `~/.local/lib/blip-cli/blip-core` on both platforms. On macOS, `BLIP_LIB` defaults to `~/.local/lib/blip-cli/BlipKit.framework/Versions/A/BlipKit`. These select the native host and library. Use the CLI installer to retain the complete signed framework. `BLIP_SOCK` selects the socket on either platform, and its folder becomes the engine's cache directory. For the CLI to start the engine, name the socket `sock` and put it in a folder you own that is not group- or world-writable, such as one made with `mkdir -m 700`, not `/tmp`. Otherwise startup fails.

## Install the skill manually

Install into the directory your agent scans for skills, using `cli-for-blip` as the folder name. Run one of these examples on the machine where the agent discovers skills. Execute transfers on the configured sender, within the verified platform limits above.

For Codex's default user skills directory:

```sh
mkdir -p ~/.codex/skills
git clone https://github.com/SahilUX/cli-for-blip-skill.git ~/.codex/skills/cli-for-blip
```

For Claude Code's user skills directory:

```sh
mkdir -p ~/.claude/skills
git clone https://github.com/SahilUX/cli-for-blip-skill.git ~/.claude/skills/cli-for-blip
```

For another agent or a customized skills location, clone into that agent's skills directory under `cli-for-blip`. The installed entrypoint must be `cli-for-blip/SKILL.md`. If the destination already exists, inspect it before updating or replacing it. Restart or reload your agent's skill discovery as needed.

To update a clone, use its installation path:

```sh
git -C ~/.codex/skills/cli-for-blip pull --ff-only
```

If you cloned this skill under its earlier name, rename the clone's folder from `blip-cli` to `cli-for-blip` when you update, so it matches the skill name. Then run `git remote set-url origin https://github.com/SahilUX/cli-for-blip-skill.git` in the clone.

To remove the skill, remove its clone from the agent's skills directory. This does not uninstall the CLI or Blip.

## Use the skill

Ask your agent to use `cli-for-blip`, for example:

> Use cli-for-blip to send `/absolute/path/report.pdf` to my device named "Target device".

The agent checks the available devices and passes `--to` explicitly. It reports delivery only after `send` exits successfully or a later check shows the transfer `Completed`. If a wait times out, the agent checks the existing transfer before considering a resend. If Blip's engine is not running, the agent starts it from an environment without API tokens or cloud credentials. The agent tells you it started the engine and offers `blip stop` once its own transfers finish. The CLI supports devices on your own account, not sending to other people's accounts.

See [SKILL.md](SKILL.md) for the workflow and the [CLI README](https://github.com/SahilUX/cli-for-blip#readme) for command documentation. The CLI uses undocumented app internals and can break after Blip updates.

## Rights and attribution

[Blip](https://blip.net/) is software and a service provided by Blip Studio Inc. This project is independent and unofficial. It is not affiliated with, endorsed by, or sponsored by Blip Studio Inc.

The [MIT license](LICENSE) applies to this repository's original skill and documentation text, copyright 2026 Sahil Saleem. It grants no rights to Blip's proprietary software, services, content, or trademarks. This repository includes no Blip logos, app binaries, or decompiled code. The separate CLI has its own MIT license and [legal provenance notice](https://github.com/SahilUX/cli-for-blip/blob/main/LEGAL.md).

Blip's [terms of service](https://blip.net/terms-of-service), last reviewed November 26, 2026 and marked updated January 18, 2025, reserve Blip's trademark and content rights, restrict trademark use without permission, and grant limited intended-purpose use. They require a Business plan for commercial activities.

The project was renamed to CLI for Blip on November 26, 2026, because those terms restrict use of Blip's trademarks. The new name still contains the word Blip, so the rename alone does not establish permission.

The CLI runs Blip's proprietary native engine, taken from the official archive, through an undocumented hosting interface, and relies on reverse-engineered, undocumented local RPC behavior. Permission for that reverse engineering, for running the engine outside Blip's app, for the RPC use, or for trademark use has not been established by this project. Attribution and the MIT license do not resolve those questions or establish legal clearance. Review Blip's current terms before use.
