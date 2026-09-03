# Superpowers for Cursor

Complete guide for using Superpowers with [Cursor](https://cursor.com).

## Installation

In Cursor Agent chat:

```text
/add-plugin superpowers
```

Or search for "superpowers" in the plugin marketplace.

Start a new chat after installing — the bootstrap loads at session start.

## How It Works

The Cursor manifest is `.cursor-plugin/plugin.json`. Unlike Claude Code's
manifest, it declares both paths explicitly:

1. `"skills": "./skills/"` — the shared skills tree.
2. `"hooks": "./hooks/hooks-cursor.json"` — Cursor's own hook config.

`hooks/hooks-cursor.json` is deliberately *not* shaped like Claude Code's
`hooks/hooks.json`. Cursor's schema uses `"version": 1`, a lowercase
`sessionStart` key, and a relative command, and it omits the
`matcher`/`type`/`async` fields Claude Code uses:

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      { "command": "./hooks/run-hook.cmd session-start" }
    ]
  }
}
```

The command runs `hooks/run-hook.cmd` (a polyglot batch/shell wrapper), which
dispatches to `hooks/session-start`. That script reads
`skills/using-superpowers/SKILL.md`, wraps it in `<EXTREMELY_IMPORTANT>` tags,
and prints Cursor's hook JSON shape — `{ "additional_context": "..." }`,
snake_case and top-level.

Cursor is detected by `CURSOR_PLUGIN_ROOT` being set, and that branch is checked
**first** because Cursor may also set `CLAUDE_PLUGIN_ROOT`. Claude Code reads
both `additional_context` and `hookSpecificOutput` without de-duplicating, so the
script emits exactly one field per harness.

## Tool Mapping

None needed. Cursor's tool surface is Claude Code–compatible, so the actions
skills describe ("invoke a skill", "dispatch a subagent", "create a todo") map
onto Cursor's native tools directly and no
`references/cursor-tools.md` file is shipped.

## Verifying the install

Smoke check, in a fresh chat:

```text
Tell me about your superpowers
```

Full check — in a clean session, send exactly:

```text
Let's make a react todo list
```

A working install triggers `brainstorming` before any code is written.

## Updating

Reinstall from the marketplace with `/add-plugin superpowers` and start a new
chat.

## Troubleshooting

### Skills never trigger

1. Confirm the plugin is installed and enabled in Cursor's plugin UI.
2. Start a new chat — the hook fires at session start only.
3. Run the hook by hand from the installed plugin directory; with
   `CURSOR_PLUGIN_ROOT` set it must print `additional_context`:

   ```bash
   CURSOR_PLUGIN_ROOT="$PWD" ./hooks/run-hook.cmd session-start
   ```

If that command prints `hookSpecificOutput` instead, `CURSOR_PLUGIN_ROOT` was not
visible to the hook — Cursor is not passing its plugin-root variable through.

## Cross-platform notes

Windows support comes from `hooks/run-hook.cmd`: `cmd.exe` runs its batch half,
which locates `bash` (Git for Windows, then `bash` on `PATH`) and runs the
extensionless hook script, exiting cleanly if no bash exists. See
[docs/windows/polyglot-hooks.md](windows/polyglot-hooks.md).

## Tests

- `tests/hooks/test-session-start.sh` — asserts the Cursor branch emits
  `additional_context` and that it contains the bootstrap.

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
- Cursor docs: https://cursor.com/docs
