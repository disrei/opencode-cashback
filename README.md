# opencode-plugin-snip

`opencode-plugin-snip` is an OpenCode plugin package that does two things:

1. Compresses chat history deterministically before it is sent to the LLM.
2. Shows a live `snip` indicator in the TUI with per-session saved-character stats.

The goal is to reduce prompt size without using another model and without changing old messages nondeterministically.

## Features

- Deterministic compression only. No LLM-based rewriting.
- Removes framework control events such as `[step-start]`, `[step-finish]`, and `[reasoning]` when they appear in framework-event form.
- Preserves `<system-reminder>...</system-reminder>` blocks.
- Normalizes tool output in `max` and `max++` modes.
- Supports three modes: `pro`, `max`, `max++`.
- Tracks saved characters per session, not globally.
- New sessions start from `0.0k`.
- Adds a bright yellow `snip` label to `home_prompt_right` and `session_prompt_right`.

## Install

Install from npm through OpenCode:

```bash
opencode plugin opencode-plugin-snip
```

If you want to install to the global OpenCode config scope, use:

```bash
opencode plugin opencode-plugin-snip --global
```

OpenCode detects the plugin package from these exports:

- `exports["./server"]`
- `exports["./tui"]`

## Configuration

After install, OpenCode writes plugin entries into `opencode.json` and `tui.json`.

The server plugin supports these options:

```json
{
  "mode": "max",
  "logEnabled": false,
  "logPath": "C:/path/to/opencode-llm.log",
  "toolMaxLines": 40,
  "toolMaxChars": 4000
}
```

### Modes

`pro`

- Smallest behavior change.
- Keeps tool payloads mostly intact.

`max`

- Default mode.
- Removes framework noise.
- Normalizes tool payloads.

`max++`

- Same as `max`, plus truncates tool output using `toolMaxLines` and `toolMaxChars`.

## TUI behavior

The TUI plugin shows a right-side label such as:

```text
snip max 12.4k
```

Behavior:

- The mode label comes from the server plugin config.
- The number is the cumulative saved characters for the current session only.
- A brand-new session starts at `0.0k`.

## Files written by the plugin

The plugin writes per-user runtime data under the OpenCode config directory:

- `~/.config/opencode/snip-stats.json`

Optional log output defaults to:

- `~/opencode-llm.log`

## How it works

Server side:

- Hooks `experimental.chat.system.transform`
- Hooks `experimental.chat.messages.transform`
- Compresses message parts deterministically
- Updates per-session saved-character stats

TUI side:

- Registers `home_prompt_right`
- Registers `session_prompt_right`
- Polls the stats file and renders live saved-character counts

## Local development

You can also use this package source directly as a local plugin during development.

Example `opencode.json`:

```json
{
  "plugin": [
    [
      "D:/Github/Opencode_plugin/src/server.js",
      {
        "mode": "max",
        "logEnabled": false,
        "toolMaxLines": 40,
        "toolMaxChars": 4000
      }
    ]
  ]
}
```

Example `tui.json`:

```json
{
  "$schema": "https://opencode.ai/tui.json",
  "plugin": [
    "D:/Github/Opencode_plugin/src/tui.tsx"
  ]
}
```

## Publish to GitHub

This directory is ready to become a GitHub repository.

Typical steps:

```bash
git init
git add .
git commit -m "Initial plugin package"
gh repo create opencode-plugin-snip --public --source . --remote origin --push
```

## Publish to npm

After the GitHub repo is ready and the package name is available:

```bash
npm publish --access public
```

If the package name is already taken, rename the `name` field in `package.json` first.

## Notes

- Do not publish your personal `opencode.json` with provider credentials.
- This package only contains the plugin code and documentation.
- The stats file is intentionally session-scoped so session totals do not bleed into each other.
