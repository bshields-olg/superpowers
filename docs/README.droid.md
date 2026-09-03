# Superpowers for Factory Droid

Complete guide for using Superpowers with [Factory](https://factory.ai)'s Droid CLI.

## Installation

Register this repository as a marketplace, then install from it:

```bash
droid plugin marketplace add https://github.com/obra/superpowers
droid plugin install superpowers@superpowers
```

Start a new session after installing.

## How It Works

Droid needs **no Droid-specific files in this repository**. It consumes the
Claude Code plugin artifacts through its own installer:

- `.claude-plugin/plugin.json` — the plugin manifest.
- `skills/` — the shared, harness-agnostic skills tree.
- `hooks/hooks.json` + `hooks/run-hook.cmd` + `hooks/session-start` — the
  session-start bootstrap.

That is deliberate. Some "new harnesses" are really an existing integration under
a different installer, and the correct port in that case adds nothing but install
instructions. See [docs/porting-to-a-new-harness.md](porting-to-a-new-harness.md),
Part 2, "You may not need a new directory at all".

## Tool Mapping

None shipped, for the same reason as Claude Code: the action vocabulary the skills
use maps onto a Claude Code–compatible tool surface, so there is no
`references/droid-tools.md`.

## Verifying the install

Smoke check, in a fresh session:

```text
Tell me about your superpowers
```

If the model does not know it has superpowers, the bootstrap is not reaching it —
see Troubleshooting below.

Full check — in a clean session, send exactly:

```text
Let's make a react todo list
```

A working install triggers `brainstorming` before any code is written.

## Updating

Reinstall from the registered marketplace:

```bash
droid plugin install superpowers@superpowers
```

Start a new session afterwards.

## Troubleshooting

### Skills never trigger

Because this integration rides Claude Code's artifacts, the failure is almost
always the bootstrap:

1. Confirm the plugin is installed and enabled in `droid plugin list`.
2. Start a fresh session.
3. From the installed plugin directory, run the hook by hand and confirm it
   prints JSON containing the bootstrap:

   ```bash
   ./hooks/run-hook.cmd session-start
   ```

   With no harness environment variables set, the script emits the SDK-standard
   `{ "additionalContext": "..." }` shape. If Droid expects a different field or
   nesting, the bootstrap will not inject — please open an issue with the details
   so a Droid branch can be added to `hooks/session-start`.

### Skills are listed but nothing loads them

Ask the model to load a skill by name (for example `brainstorming`). If it can
load skills on request but never does so on its own, the skill index is present
but the bootstrap is missing — same diagnosis as above.

## Tests

There are no Droid-specific tests, because there are no Droid-specific files.
`tests/hooks/test-session-start.sh` covers the hook this integration reuses.

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
- Factory docs: https://docs.factory.ai
