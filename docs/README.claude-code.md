# Superpowers for Claude Code

Complete guide for using Superpowers with [Claude Code](https://claude.com/claude-code).

## Installation

Superpowers is available from [Anthropic's official plugin marketplace](https://claude.com/plugins/superpowers).

```text
/plugin install superpowers@claude-plugins-official
```

The Superpowers marketplace carries Superpowers plus related plugins:

```text
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

Restart Claude Code (or start a new session) after installing — the bootstrap
loads at session start.

## How It Works

The Claude Code manifest lives at `.claude-plugin/plugin.json`. It sets no
`skills` or `hooks` paths because Claude Code auto-discovers both by convention:

1. `skills/` — every Superpowers skill, read directly from the installed plugin.
   No copies and no symlinks.
2. `hooks/hooks.json` — a `SessionStart` hook with the matcher
   `startup|clear|compact`, so the bootstrap re-injects after `/clear` and after
   compaction as well as at startup.

The hook command is `"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd" session-start`.
`hooks/run-hook.cmd` is a polyglot batch/shell wrapper (see
[Cross-platform notes](#cross-platform-notes)) that dispatches to
`hooks/session-start`, which:

- reads `skills/using-superpowers/SKILL.md` in full,
- wraps it in `<EXTREMELY_IMPORTANT>` tags with the "You have superpowers" preamble,
- and prints Claude Code's hook JSON shape:
  `{ "hookSpecificOutput": { "hookEventName": "SessionStart", "additionalContext": "..." } }`.

That injected skill is the whole integration: it is what makes the agent check
for a relevant skill before it acts. Everything else in `skills/` is loaded on
demand from there.

The hook script picks its output shape from the environment. Claude Code is
detected by `CLAUDE_PLUGIN_ROOT` being set while `COPILOT_CLI` is not — Claude
Code reads both `additional_context` and `hookSpecificOutput` without
de-duplicating, so exactly one field is emitted.

## Tool Mapping

None needed. Skills describe actions ("dispatch a subagent", "create a todo",
"invoke a skill") and Claude Code's native tools — `Skill`, `Task`, `TodoWrite`,
`Read`/`Write`/`Edit`, `Bash`, `Grep`, `Glob`, `WebFetch` — are the vocabulary
those actions were written against, so there is no
`skills/using-superpowers/references/claude-code-tools.md`. Harnesses whose tool
names differ ship a reference file instead.

## Verifying the install

Quick smoke check — ask in a fresh session:

```text
Tell me about your superpowers
```

If the bootstrap injected, the agent knows it has skills and can name them.

Full check — in a clean session, send exactly:

```text
Let's make a react todo list
```

A working install triggers the `brainstorming` skill before any code is written.

## Updating

Claude Code updates marketplace plugins for you. To force an update, use
`/plugin` and reinstall Superpowers from the marketplace you installed it from,
then start a new session.

## Troubleshooting

### Skills never trigger

The bootstrap is not loading. Check, in order:

1. `/plugin` shows Superpowers installed and enabled.
2. The session is new — the hook only runs on `startup`, `/clear`, and compaction.
3. Run the hook by hand from the installed plugin directory and confirm it prints
   JSON containing `hookSpecificOutput`:

   ```bash
   CLAUDE_PLUGIN_ROOT="$PWD" ./hooks/run-hook.cmd session-start
   ```

### The bootstrap appears twice

Only one context field may be emitted per harness. If you have modified
`hooks/session-start`, check that your branch emits either
`additional_context` or `hookSpecificOutput`, never both.

## Cross-platform notes

`hooks/run-hook.cmd` is valid as both a Windows batch file and a Unix shell
script. On Windows, `cmd.exe` runs the batch half, which locates `bash` (Git for
Windows first, then `bash` on `PATH`) and runs the hook script; if no bash is
found it exits cleanly, so Claude Code still works — just without injection. On
macOS and Linux the leading `:` makes the batch block a no-op.

Hook scripts are deliberately extensionless (`session-start`, not
`session-start.sh`): Claude Code's Windows handling prepends `bash` to any
command containing `.sh`, which would double-invoke. See
[docs/windows/polyglot-hooks.md](windows/polyglot-hooks.md).

## Tests

- `tests/hooks/test-session-start.sh` — asserts the hook's JSON shape per harness.
- `tests/claude-code/` — Claude Code behavior and workflow tests, plus token
  analysis helpers. See [docs/testing.md](testing.md).

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
- Claude Code docs: https://docs.claude.com/en/docs/claude-code
