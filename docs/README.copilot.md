# Superpowers for GitHub Copilot CLI

Complete guide for using Superpowers with [GitHub Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli).

## Requirements

Copilot CLI **v1.0.11 or newer**. That release added support for
`additionalContext` in `sessionStart` hook output, which is how Superpowers
injects its bootstrap. On older versions the plugin installs but the bootstrap
never reaches the model, so skills do not auto-trigger.

## Installation

```bash
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace
```

Start a new session after installing.

## How It Works

Copilot CLI has no dedicated manifest in this repo — it consumes the Claude
Code plugin layout (`skills/` plus `hooks/hooks.json`) and differs only in the
JSON shape it reads back from the hook.

At session start the hook runs `hooks/run-hook.cmd session-start`, which
dispatches to `hooks/session-start`. That script reads
`skills/using-superpowers/SKILL.md`, wraps it in `<EXTREMELY_IMPORTANT>` tags,
and then chooses its output shape from the environment:

- Copilot CLI sets `COPILOT_CLI`, so the script falls through to the SDK-standard
  shape: `{ "additionalContext": "..." }` — top-level, camelCase.
- Claude Code's nested `hookSpecificOutput` shape is used only when
  `CLAUDE_PLUGIN_ROOT` is set *and* `COPILOT_CLI` is not, because Copilot CLI
  sets both.

Emitting the wrong field is a silent failure: the hook exits 0 and the model
simply never learns it has superpowers.

## Tool Mapping

None needed. Copilot CLI's tool surface is Claude Code–compatible — it has a
native `Skill`-style tool, file read/write/edit, shell, search, and web tools —
so the action vocabulary in the skills resolves without a translation file. The
old `references/copilot-tools.md` was removed once nothing harness-specific was
left in it.

One platform note that has bitten users: prefer Copilot CLI's own backgrounding
guidance on Windows over shell tricks when a skill asks you to run something in
the background.

## Verifying the install

Smoke check, in a fresh session:

```text
Tell me about your superpowers
```

Full check — in a clean session, send exactly:

```text
Let's make a react todo list
```

A working install triggers `brainstorming` before any code is written.

## Updating

```bash
copilot plugin install superpowers@superpowers-marketplace
```

Reinstalling from the marketplace picks up the latest release. Start a new
session afterwards.

## Troubleshooting

### Skills never trigger

1. Check your Copilot CLI version — anything before v1.0.11 cannot ingest
   `additionalContext`.
2. Confirm the plugin is installed and enabled.
3. Run the hook by hand from the installed plugin directory; it must print a
   top-level `additionalContext`:

   ```bash
   COPILOT_CLI=1 ./hooks/run-hook.cmd session-start
   ```

### The hook prints `hookSpecificOutput`

`COPILOT_CLI` was not set in the hook's environment, so the script took the
Claude Code branch. Copilot CLI sets that variable itself; if it is missing,
you are probably running the hook outside Copilot CLI.

## Cross-platform notes

`hooks/run-hook.cmd` is a polyglot batch/shell wrapper so the same hook works on
Windows, macOS, and Linux; hook scripts stay extensionless on purpose. See
[docs/windows/polyglot-hooks.md](windows/polyglot-hooks.md).

## Tests

- `tests/hooks/test-session-start.sh` — asserts the Copilot CLI branch emits a
  top-level `additionalContext` containing the bootstrap.

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
- Copilot CLI docs: https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli
