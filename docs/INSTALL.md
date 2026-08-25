# Install and use

> **Status: DEVELOPER PREVIEW**

Muffin is not yet distributed as a public package. These instructions apply to
authorised evaluators and contributors with access to a `muffin-agent` source
checkout.

## Requirements

- Node.js 22 or newer.
- A supported model provider: Anthropic or an OpenAI-compatible endpoint.
- A user account able to write to its own home and `~/.local/bin`.
- macOS or Linux for the current source installer path.

The supported public platform and installer matrix is not frozen yet.

## Install from an authorised checkout

From the repository root:

```sh
./install.sh
```

The installer installs dependencies, builds the CLI and links a user-level
command without using `sudo`. It normally installs `muffin`; if that name is
already owned by another program, it may use `muffin-agent` instead and reports
the chosen command.

Then run the setup flow:

```sh
muffin init
```

The interactive flow asks for provider configuration and reads the secret using
hidden input. Do not put API keys in command-line arguments, shell history or
public configuration files.

## First use

```sh
muffin                 # interactive agent
muffin run "<goal>"    # one goal, then exit
muffin doctor          # inspect setup and runtime health
```

Useful inspection commands include:

```sh
muffin config
muffin memory stats
muffin memory search "<query>"
muffin vault ls
muffin jobs list
muffin surface list
muffin gateway status
muffin mcp list --verify
muffin prompt show
```

## Optional Telegram surface

Telegram is a current dogfood surface, not a general-release onboarding path.
After configuring the required token, the operator flow is:

```sh
muffin surface enable telegram
muffin surface list
```

The exact pairing and owner-binding flow may change while DAY-1 hardening is in
progress.

## Resident operation

Scheduled and waiting work can be kept alive by Muffin's gateway. The CLI can
inspect, stop and install the current platform service:

```sh
muffin gateway status
muffin gateway install --write
```

Review the command output before enabling a resident service on a real machine.

## Development

From the source checkout:

```sh
npm run build
npm test
npm run compile
```

These commands validate types, run the test suite and produce the executable
build. Passing unit tests alone is not evidence that every real-process Gate is
closed.

## Uninstall and data

Removing the launcher and removing owner data are separate operations. The
installer can remove the launcher it created; the `muffin uninstall` command
removes the active Muffin home only after explicit confirmation.

Back up owner data before destructive testing. Upgrade, migration and recovery
paths are active hardening areas and are not yet a stable public contract.
