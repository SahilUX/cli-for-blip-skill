---
name: blip-cli
description: Send files and folders to devices on the user's own Blip account through the separately installed blip-cli on Linux. Use for Blip sends, device discovery, and transfer checks. Other recipients and installation of Blip itself are outside this skill's send workflow.
---

# Use blip-cli

This skill uses the independent [blip-cli](https://github.com/SahilUX/blip-cli) project, normally installed as `blip`. It contains no CLI or Blip app. Read [installation and prerequisites](README.md#prerequisites) when setup is missing. CLI access is currently restricted because its repository is private.

## Check the environment

Run commands on the user's configured Linux sender as the account that owns the Blip session. Discover that environment from the user or project configuration rather than assuming a host, username, device, or service.

Check `command -v blip`, `node --version`, and `blip --help` before using the CLI. Confirm the help identifies the expected CLI, since `blip` can also launch the official app. If the CLI has a custom executable name, use that verified command throughout.

The sender needs Node.js 22+, the separately installed CLI, the official Blip Linux app, and a signed-in Blip account. Headless startup uses Xvfb and D-Bus. `curl` and `git` are installation prerequisites. The recipient needs Blip on the same account, open on phones or awake on computers.

`blip status`, `blip devices`, and `blip transfers` can start the background app. `status` can exit 0 while signed out, so read its message. If signed out, have the user complete login through the CLI's documented interactive flow. Login, logout, installation, and service changes need their own task authorization.

## Send the requested files

1. Run `blip devices` and resolve the intended recipient among the user's own devices. Match the full name, ignoring case. The sender is excluded by `send`. If names are ambiguous, have the user rename a device before sending. This CLI does not send to other people's accounts.
2. Resolve the requested paths on the sender. Verify that each exists and that a folder's contents fit the requested scope. Files are read in place, so keep them available until completion.
3. Send with an explicit recipient every time. Quote paths and names as shell arguments. Use absolute paths so a filename beginning with `-` cannot become an option.

   ```sh
   blip send /absolute/path/report.pdf /absolute/path/photos/ --to "Target device"
   ```

   `--to` avoids the CLI's fallback to `BLIP_TO`, a remembered recipient, or the only other device. It also updates the remembered recipient.

4. Wait for the command to exit. Progress goes to stderr. Report delivery only when `send` exits 0, which means `Completed`. Include the recipient and files in the result.

## Handle incomplete transfers

- Exit 1 means an error or cancellation. Inspect the reported error before deciding what to do next.
- Exit 2 means invalid usage. Correct the command using `blip --help`.
- Exit 3 means the wait expired. The default is 300 seconds, configurable with `BLIP_TIMEOUT_S`. A timeout does not cancel the transfer.

For an interrupted or timed-out send, check the existing transfer before retrying:

```sh
blip transfers <transfer-id>
```

If the id was lost, use `blip transfers` to find it. An unfinished transfer may still deliver later. Avoid automatic resends that could duplicate delivery. For a subsequent authorized send, choose a longer `BLIP_TIMEOUT_S` when the file size warrants it.

For startup or connection failures, consult the [CLI troubleshooting guide](https://github.com/SahilUX/blip-cli#troubleshooting). Use the CLI's summaries and share only necessary, redacted diagnostics. Never read `~/.local/share/net.blip.Blip/state.dat` or request or print raw `GetState` output. Both contain session credentials. App logs and status output can also contain account or device details.

## Compatibility and rights

The CLI depends on undocumented app internals originally examined in Blip 1.2.6. An app update can break compatibility. Stop on unexpected behavior and report the command, version, and redacted error instead of inventing RPC calls.

[Blip](https://blip.net/) is provided by Blip Studio Inc. This skill is independent and unofficial, with no affiliation or endorsement. Its MIT license covers this repository's original text, not Blip's proprietary software, services, or trademarks. Read the [rights notice](README.md#rights-and-attribution) for the terms and unresolved permission questions.
