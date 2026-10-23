---
name: blip-cli
description: Send files and folders to devices on the user's own Blip account through the separately installed blip-cli on Linux or the macOS preview. Use for Blip sends, device discovery, and transfer checks. Other recipients and installation of Blip itself are outside this skill's send workflow.
---

# Use blip-cli

This skill uses the independent [blip-cli](https://github.com/SahilUX/blip-cli) project, normally installed as `blip`. It contains no CLI or native engine. Read [installation and prerequisites](README.md#prerequisites) when setup is missing. CLI access is currently restricted because its repository is private.

## Check the environment

Run commands on the user's configured sender as the account that owns the Blip session. Discover that environment from the user or project configuration rather than assuming a host, username, device, or service.

Check `command -v blip`, `node --version`, and `blip --help` before using the CLI. Confirm the help identifies the expected CLI, since `blip` can also launch the official app. If the CLI has a custom executable name, use that verified command throughout.

The sender needs Node.js 22+, the separately installed CLI and its native runtime, and a signed-in Blip account. The CLI runs a small C host directly, without a GUI, Java, Xvfb, or D-Bus. Read the [platform prerequisites](README.md#prerequisites) when setup is missing or platform support is uncertain. Linux x86-64 with glibc has verified existing-session transfers; aarch64 and fresh email-code login remain experimental. macOS remains a preview. Apple Silicon has verified real email-code login, authenticated file and nested-folder transfers to Linux and the Mac desktop app, and native restart with the signed-in device preserved. Real-service logout and unattended or SSH operation on macOS remain unverified. Intel is experimental. The recipient needs Blip on the same account, open on phones or awake on computers.

On macOS, use the CLI's separate profile. A desktop sign-in does not authenticate the CLI. Preserve the desktop app's credential files and process; the CLI does not read or migrate those files or stop the desktop app. The framework may use Apple system storage. Do not infer that all credentials live under the CLI data directory. Read the [macOS profile paths](README.md#macos-preview-profile) for setup or diagnostics.

`blip status` inspects the engine without starting it and exits 1 if it is stopped. Every command except `status` and `stop` starts the native engine when needed. `status` can exit 0 while signed out, so read its message. `blip stop` stops only an instance started by the CLI. On Linux, an already answering desktop or service instance is reused; avoid starting a second instance while another is starting. If signed out, have the user complete login through the CLI's documented interactive flow. Login, logout, installation, and service changes need their own task authorization.

## Send the requested files

1. Run `blip devices` and resolve the intended recipient among the user's own devices. Match the full name, ignoring case. The sender is excluded by `send`. If names are ambiguous, use the intended device ID from `blip devices`; clarify the recipient if the ID cannot be resolved from the request. This CLI does not send to other people's accounts.
2. Resolve the requested paths on the sender. Verify that each exists and that a folder's contents fit the requested scope, including hidden files such as `.env` and `.git`. Files are read in place, so keep them available until completion.
3. Send with an explicit recipient every time. Quote paths and names as shell arguments. Use absolute paths so a filename beginning with `-` cannot become an option.

   ```sh
   blip send /absolute/path/report.pdf /absolute/path/photos/ --to "Target device"
   ```

   `--to` avoids the CLI's fallback to `BLIP_TO`, a remembered recipient, or the only other device. It also updates the remembered recipient.

4. Wait for the command to exit. Progress goes to stderr. Report delivery only when `send` exits 0, which means `Completed`. Include the recipient and files in the result.

## Handle incomplete transfers

- Exit 1 means an error or cancellation. Inspect the reported error before deciding what to do next.
- Exit 2 means invalid usage. Correct the command using `blip --help`.
- Exit 3 means the wait expired or contact with Blip was lost after transfer creation. The default is 300 seconds, configurable with `BLIP_TIMEOUT_S`. A timeout does not cancel the transfer. Exit 130 or 143 means the wait was interrupted or terminated; the transfer can still deliver.

For an interrupted or timed-out send, check the existing transfer before retrying:

```sh
blip transfers <transfer-id>
```

If the id was lost, use `blip transfers` to find it. An unfinished transfer may still deliver later. Avoid automatic resends that could duplicate delivery. For a subsequent authorized send, choose a longer `BLIP_TIMEOUT_S` when the file size warrants it.

For startup or connection failures, consult the [CLI troubleshooting guide](https://github.com/SahilUX/blip-cli#troubleshooting). Use the summaries from `blip status`, `blip devices`, and `blip transfers`, and share only necessary, redacted diagnostics. Never read Blip's `state.dat` or request or print raw `GetState` output. Both contain session credentials. App logs and status output can also contain account or device details.

## Compatibility and rights

The installer pins the native engine to Blip 1.2.6 on Linux and Blip 1.2.3 on macOS because its hosting interface and RPC protocol are undocumented. `BLIP_APP` selects the native host executable, not a desktop app; `BLIP_LIB` selects the native library. An incompatible engine can break startup or RPC compatibility. Stop on unexpected behavior and report the command, version, and redacted error instead of inventing RPC calls.

[Blip](https://blip.net/) is provided by Blip Studio Inc. This skill is independent and unofficial, with no affiliation or endorsement. Its MIT license covers this repository's original text, not Blip's proprietary software, services, or trademarks. Read the [rights notice](README.md#rights-and-attribution) for the terms and unresolved permission questions.
