# Superpowers for Antigravity

Complete guide for using Superpowers with Antigravity's `agy` CLI.

## Installation

```bash
agy plugin install https://github.com/obra/superpowers
```

Antigravity runs the plugin's session-start hook, so Superpowers is active from
the first message. Reinstall with the same command to update.

## How It Works

`agy` installs the existing plugin **directly** — there is no Antigravity-specific
manifest, injector, or scaffold in this repository. The install brings along:

- `skills/` — the shared, harness-agnostic skills tree, which `agy` copies in and
  scans.
- `hooks/hooks.json` + `hooks/run-hook.cmd` + `hooks/session-start` — the
  session-start hook that reads `skills/using-superpowers/SKILL.md`, wraps it in
  `<EXTREMELY_IMPORTANT>` tags, and prints it as the session's additional context.

What *is* Antigravity-specific is the tool mapping, because two of the actions the
skills describe have no obvious counterpart on this harness (see below).

## Loading skills

Antigravity has native skill *discovery* but no Claude Code–style `Skill` tool.
The sanctioned way to load a skill here is to **read its `SKILL.md` with the
file-read tool when the skill applies**. That honors
`using-superpowers`'s "never read skill files manually" rule rather than breaking
it: the rule means "do not bypass your platform's skill-loading mechanism", and on
a harness with no skill tool, reading the file *is* the mechanism.

## Tool Mapping

[`skills/using-superpowers/references/antigravity-tools.md`](../skills/using-superpowers/references/antigravity-tools.md),
linked from `using-superpowers`'s Platform Adaptation section:

| Action skills request | Antigravity equivalent |
|---|---|
| Dispatch a subagent | `invoke_subagent` with a built-in `TypeName` — `self` for full-capability work, `research` for read-only |
| Task tracking ("create a todo", "mark complete") | a **task artifact** — `write_to_file` with `IsArtifact: true` and `ArtifactMetadata.ArtifactType: "task"` |

### Task tracking is an artifact, not `manage_task`

Antigravity has **no todo tool**. `manage_task` manages *background processes*
(`list`/`kill`/`status`/`send_input`) and is not a checklist. When a skill asks for
a todo list:

1. At the start of any multi-step task, write a markdown checklist as a task
   artifact (`write_to_file` with `IsArtifact: true`,
   `ArtifactMetadata.ArtifactType: "task"`) listing every step of the plan.
2. Mark steps done (`- [x]`) with `replace_file_content` or
   `multi_replace_file_content` as you go, and update the list when the plan
   changes.
3. Re-read it before each step once the conversation gets long — it is the source
   of truth for what remains.

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
agy plugin install https://github.com/obra/superpowers
```

The same command reinstalls over the top.

## Troubleshooting

### The model does not know it has superpowers

The session-start hook is not delivering the bootstrap.

1. Reinstall and confirm `agy` reports the plugin's components as installed.
2. Confirm the installed tree still contains `skills/using-superpowers/SKILL.md`
   and `hooks/session-start`. A plugin installer typically copies only components
   it recognizes and discards the rest, so a partial install is a real
   possibility — and it looks exactly like "skills never trigger".
3. From the installed plugin directory, run the hook by hand and confirm it prints
   JSON containing the bootstrap:

   ```bash
   ./hooks/run-hook.cmd session-start
   ```

### The model tries to use `manage_task` for a todo list

That is the wrong tool — see the task-artifact section above. The mapping is
loaded from `references/antigravity-tools.md`; if the model has not read it, point
it there.

## Tests

- `tests/antigravity/test-antigravity-tools.sh` — asserts the mapping documents
  `invoke_subagent` with the `self`/`research` types, `write_to_file` /
  `replace_file_content`, task tracking as a `task` artifact, and that
  `using-superpowers/SKILL.md` links the mapping. CI-safe: it does not require
  `agy` to be installed.

Run them with `bash tests/antigravity/run-tests.sh`.

## Getting Help

- Report issues: https://github.com/obra/superpowers/issues
- Main documentation: https://github.com/obra/superpowers
