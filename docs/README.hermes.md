# Superpowers for Hermes Agent

Complete guide for using Superpowers with Hermes Agent.

## Installation

```bash
hermes plugins install obra/superpowers --enable
```

Restart any active Hermes sessions after installing — the bootstrap is injected
on a session's first turn.

## How It Works

Hermes is a **Python plugin** integration. `.hermes-plugin/plugin.yaml` is the
manifest:

```yaml
name: superpowers
version: 6.3.0
description: Superpowers skills and workflow bootstrap for Hermes Agent
author: obra
provides_hooks:
  - pre_llm_call
```

`.hermes-plugin/__init__.py` does the work in its `register(ctx)` function:

1. **Locates the skills tree.** Both supported install layouts are handled: a git
   clone (where `.hermes-plugin/` and `skills/` are siblings, so it resolves
   `../skills`) and a flattened install (where `skills/` sits next to the module).
   If neither matches it raises loudly rather than skipping the bootstrap — a
   bootstrap that silently no-ops is how a broken install masquerades as a working
   one.
2. **Registers every skill** with Hermes' native loader via
   `ctx.register_skill(name, Path(skill_md))`, so `skill_view` can load them on
   demand. (`register_skill` requires a `pathlib.Path`; a `str` raises
   `AttributeError` and Hermes silently disables the whole plugin.)
3. **Injects the bootstrap** from a `pre_llm_call` hook that returns
   `{"context": bootstrap}` on the first turn only. This is the documented
   injection path: `on_session_start` return values are ignored and
   `ctx.inject_message` refuses to run from that hook. The context is appended to
   the first turn's user message.

The bootstrap itself is assembled from `using-superpowers/SKILL.md` (frontmatter
stripped) wrapped in `<EXTREMELY_IMPORTANT>`, with the marker
`superpowers:using-superpowers bootstrap for hermes`, a note that the skill is
already loaded and must not be re-loaded, instructions for loading skills on
Hermes, the absolute skills directory, and the full Hermes tool mapping.

### Known limitation

Hermes has **no post-compaction hook**. The bootstrap goes in on the first turn
only, so a very long session that compacts over that first turn loses it. If
skills stop triggering, start a fresh session.

## Loading skills

```text
skill_view("superpowers:brainstorming")
```

If a namespaced lookup returns "not found" (the skill may not be in the catalog
until the plugin fully registers it), read the file directly instead:

```text
read_file("<skills-dir>/brainstorming/SKILL.md")
```

The injected bootstrap includes the resolved absolute skills directory, so the
model always has the real path. This file-read fallback is the same sanctioned
mechanism other harnesses without native skill loading use.

## Tool Mapping

[`skills/using-superpowers/references/hermes-tools.md`](../skills/using-superpowers/references/hermes-tools.md)
is appended to the bootstrap in full, so the agent always has it:

| Action skills request | Hermes tool |
|---|---|
| Read a file | `read_file` |
| Create a file | `write_file` |
| Edit a file (targeted patch) | `patch` |
| Run a shell command | `terminal` |
| Search file contents | `search_files` |
| Find files by name | `terminal` with `find` |
| Fetch a URL | `web_extract(urls=[...])` |
| Search the web | `web_search(query=...)` |
| Dispatch a subagent | `delegate_task(goal=..., context=..., toolsets=[...], role="leaf")` |
| Task tracking | `todo` |
| Invoke a skill | `skill_view("skill-name")` |

Also covered there: when a skill says "your instructions file", on Hermes that is
`AGENTS.md` in the project directory or `~/.hermes/SOUL.md` globally; and if
`delegate_task` is unavailable, do the work inline rather than inventing tool
calls.

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

Reinstall from the repository:

```bash
hermes plugins install obra/superpowers --enable
```

Restart your sessions afterwards. `.hermes-plugin/plugin.yaml` is registered in
`.version-bump.json`, so its version tracks the rest of the plugin.

## Troubleshooting

### The plugin is installed but nothing happens

Hermes disables a plugin silently when `register()` raises. Check Hermes' plugin
diagnostics/logs for the `superpowers plugin: cannot find the skills/ tree` error
— that means neither install layout matched, and a reinstall with
`hermes plugins install obra/superpowers` is the fix.

### `skill_view("superpowers:brainstorming")` says not found

Drop the namespace or read the `SKILL.md` directly from the skills directory
printed in the bootstrap. Both are documented in the mapping above.

### Skills stopped triggering mid-session

See the compaction limitation above: start a fresh session.

## Tests

- `tests/hermes/test_plugin.py` and `tests/hermes/test_bootstrap.py` — pytest
  suites covering skill registration and bootstrap assembly/injection.

Run them with `pytest tests/hermes`.

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
