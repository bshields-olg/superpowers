# Superpowers for Devin CLI

Complete guide for using Superpowers with [Devin CLI](https://devin.ai).

## Installation

```bash
devin plugins install obra/superpowers
```

Start a new session after installing.

## How It Works

`devin plugins install obra/superpowers` reads `.devin-plugin/plugin.json` and
auto-discovers the co-located `skills/` directory. Devin CLI surfaces every
installed skill's `name` and `description` in its system prompt at session start
and invokes them through its native `skill` tool.

That surfaced skill index *is* the bootstrap: `using-superpowers`'s own
description ("Use when starting any conversation…") is what prompts the model to
load it and then check for a relevant skill before acting. There is no hook, no
injector, and no context file to ship.

Two consequences worth knowing:

- The Devin manifest is **metadata only**. It carries no `skills`, `hooks`,
  `commands`, `sessionStart`, `contextFileName`, or `inject` fields — Devin
  supports none of them, and `tests/devin/test-devin-plugin.sh` fails if one
  appears.
- Because nothing is injected, there is no `<EXTREMELY_IMPORTANT>` wrapper and no
  re-injection after compaction. If a very long session stops triggering skills,
  start a fresh one.

## Tool Mapping

None shipped. Devin CLI's system prompt already documents its own tools —
subagent profiles, todo tracking, and question prompts — so the actions the
skills describe resolve against that, and there is no
`references/devin-tools.md`.

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
devin plugins update superpowers
```

Start a new session afterwards.

## Troubleshooting

### Skills never trigger

1. Confirm the plugin is installed and enabled in `devin plugins`.
2. Start a fresh session — the skill index is built into the system prompt at
   session start.
3. Ask the model to list the skills it can see. If Superpowers' skills are
   missing, the co-located `skills/` directory did not come along with the
   install; reinstall the plugin.

## Tests

- `tests/devin/test-devin-plugin.sh` — validates the manifest: name, version
  matching `package.json`, absence of the unsupported fields above, and that
  `.version-bump.json` keeps `.devin-plugin/plugin.json` in lockstep.

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
- Devin docs: https://docs.devin.ai
