---
name: blip-cli
description: Send files and folders to devices on the user's own Blip account through the separately installed blip-cli on Linux or the macOS preview. Use for Blip sends, device discovery, and transfer checks. Other recipients and installation of Blip itself are outside this skill's send workflow.
---

# Use blip-cli

This skill uses the independent [blip-cli](https://github.com/SahilUX/blip-cli) project, normally installed as `blip`. It contains no CLI or native engine. Read [installation and prerequisites](README.md#prerequisites) when setup is missing. CLI access is currently restricted because its repository is private.

## Check the environment

Run commands on the user's configured sender as the account that owns the Blip session. Discover that environment from the user or project configuration rather than assuming a host, username, device, or service.

Check `node --version` and locate the CLI with `command -v blip`. Non-interactive shells, such as SSH commands, cron, or services, often lack `~/.local/bin` on `PATH`, so check `~/.local/bin/blip` before concluding the CLI is missing. `blip` can also be the official app's launcher, so confirm it is this CLI without executing it: `readlink -f` on its path ends in `/blip.mjs`, or the file's second line starts with `// blip: headless command line for the Blip native engine`. Only then run `blip --help`. If the CLI has a custom executable name or path, use that verified command throughout.

The sender needs Node.js 22+, the separately installed CLI and its native runtime, and a signed-in Blip account. The CLI runs a small C host directly, without a GUI, Java, Xvfb, or D-Bus. Read the [platform prerequisites](README.md#prerequisites) when setup is missing or platform support is uncertain. Linux x86-64 with glibc has verified existing-session transfers; aarch64, fresh email-code login, and real-service logout remain experimental. macOS remains a preview. Apple Silicon has verified real email-code login, authenticated file transfers to Linux and the Mac desktop app, nested-folder transfers to the Mac desktop app, and native restart with the signed-in device preserved. Real-service logout and unattended or SSH operation on macOS remain unverified. Intel is experimental. The recipient needs Blip on the same account, open on phones or awake on computers.

On macOS, use the CLI's separate profile. A desktop sign-in does not authenticate the CLI. Preserve the desktop app's credential files and process; the CLI does not read or migrate those files or stop the desktop app. The framework may use Apple system storage. Do not infer that all credentials live under the CLI data directory. Read the [macOS profile paths](README.md#macos-preview-profile) for setup or diagnostics.

Run `blip status` before any `blip` command other than `--help`. It reads the engine's state once, never starts the engine, and exits 1 if the engine is not running. It can exit 0 while signed out, so read its message. Every command except `status` and `stop` starts the native engine when needed. On Linux, an already answering desktop or service instance is reused; avoid starting a second instance while another is starting. If signed out, have the user complete login through the CLI's documented interactive flow. Login, logout, installation, and service changes need their own task authorization.

The engine inherits the environment of the command that starts it and keeps that environment for its whole lifetime. When `blip status` exits 1, start the engine with `blip start` from an environment without API tokens or cloud credentials, for example `env -u ANTHROPIC_API_KEY -u GITHUB_TOKEN blip start`, naming the secret variables your shell actually holds. To find them, list variable names only, for example with `node -p 'Object.keys(process.env).sort().join("\n")'`, and never print their values.

A running engine makes this machine an online Blip device. It keeps running after the command exits and, depending on Blip's auto-accept setting, may save files from the user's other devices here without a prompt. If you started it, tell the user, and offer `blip stop` once no transfer you created is unfinished. `blip stop` stops only an instance started by the CLI.

## Send the requested files

1. Run `blip devices` and resolve the intended recipient among the user's own devices. Match the full name, ignoring case. The sender is excluded by `send`. If names are ambiguous, use the intended device ID from `blip devices`; clarify the recipient if the ID cannot be resolved from the request. This CLI does not send to other people's accounts.
2. Resolve the requested paths on the sender. Verify that each exists and that a folder's contents fit the requested scope, including hidden files such as `.env` and `.git`. `send` takes a folder whole and cannot exclude files, so ask before sending when hidden or credential files fall outside what the user asked for. Files are read in place, so keep them available until completion.

   `send` refuses Blip's own data folders, anything in them, and any folder that contains them, such as `~` or `/`. It fails with `refusing to send <path>: it contains Blip's own data, including your session`. On macOS it also refuses the `/System/Volumes/Data` and `/.nofollow` spellings of those paths. The check compares paths only, so a copy or hard link of that data elsewhere passes the check and still carries the session. Narrow the send to the files the user wants.

3. Send with an explicit recipient every time. Quote paths and names as shell arguments. Use absolute paths so a filename beginning with `-` cannot become an option. `send` waits up to 300 seconds by default, which can exceed your command time limit. Set `BLIP_TIMEOUT_S` below that limit, as this example does for a 120-second limit, or run `send` in the background and wait for it.

   ```sh
   BLIP_TIMEOUT_S=90 blip send /absolute/path/report.pdf /absolute/path/photos/ --to "Target device"
   ```

   `--to` avoids the CLI's fallback to `BLIP_TO`, a remembered recipient, or the only other device. It also updates the remembered recipient.

4. Wait for `send` to exit. Progress goes to stderr; keep it. The line `transfer <id> -> <device>`, printed before the wait starts, holds the transfer ID. Report delivery only when `send` exits 0, which means `Completed`, or when a later `blip transfers <id>` for the ID you captured shows `Completed`. Include the recipient and files in the result.

## Handle incomplete transfers

- Exit 1 means an error or cancellation. Inspect the reported error before deciding what to do next.
- Exit 2 means invalid usage. Correct the command using `blip --help`.
- Exit 3 means the wait expired or contact with Blip was lost after transfer creation. `BLIP_TIMEOUT_S` sets the wait. A timeout does not cancel the transfer. Exit 130 or 143 means the wait was interrupted or terminated; the transfer can still deliver.
- Handle `send` like exit 3 when your harness killed it or timed it out, or when it exited with any code other than 0, 1, or 2 after printing the `transfer <id>` line. A killed `send`, such as exit 137 after SIGKILL, prints no `stays queued` message.

For an interrupted or timed-out send, check the existing transfer before retrying:

```sh
blip transfers <transfer-id>
```

Without an ID, `blip transfers` shows only each transfer's ID, direction, and state, with no recipient, file, or time, so it cannot identify a lost transfer. If you did not capture the `transfer <id>` line, do not guess from the list and do not resend. Tell the user a transfer may still be queued and that it can be declined or cancelled only on the receiving device. An unfinished transfer may still deliver later. Check a captured ID again later. Avoid automatic resends that could duplicate delivery. For a later authorized send of a large file, choose a longer `BLIP_TIMEOUT_S` that still fits your command time limit, or run `send` in the background.

For startup or connection failures, consult the [CLI troubleshooting guide](https://github.com/SahilUX/blip-cli#troubleshooting). Use the summaries from `blip status`, `blip devices`, and `blip transfers`, and share only necessary, redacted diagnostics. Never read Blip's `state.dat` or request or print raw `GetState` output. Both contain session credentials. App logs and status output can also contain account or device details. Do not list, read, move, or delete the contents of Blip's session data directories or the desktop app's Group Container, and do not list, read, or change Blip's Keychain items. Use the directory paths only to recognize them.

## Compatibility and rights

The installer pins the native engine to Blip 1.2.6 on Linux and Blip 1.2.3 on macOS because its hosting interface and RPC protocol are undocumented. `BLIP_APP` selects the native host executable, not a desktop app; `BLIP_LIB` selects the native library. `BLIP_SOCK` selects the socket, and its folder becomes the engine's cache directory. For the CLI to start the engine, the socket file must be named `sock`, in a folder the user owns that is not group- or world-writable, such as one made with `mkdir -m 700`. `/tmp` does not work. A folder that fails this check makes startup fail with `refusing <folder> for BLIP_SOCK: Blip keeps its cache there, so it must be a folder you own that no one else can write to`. An incompatible engine can break startup or RPC compatibility. Stop on unexpected behavior and report the command, version, and redacted error instead of inventing RPC calls.

[Blip](https://blip.net/) is provided by Blip Studio Inc. This skill is independent and unofficial, with no affiliation or endorsement. Its MIT license covers this repository's original text, not Blip's proprietary software, services, or trademarks. Read the [rights notice](README.md#rights-and-attribution) for the terms and unresolved permission questions.
