# codestd

Coding conventions for humans and AI coding agents: one language-agnostic core, plus a pack per
language with its idioms, thresholds and linter settings. Packaged as an agent skill.

## Layout

| Path | Holds |
|---|---|
| `skills/codestd/SKILL.md` | The skill: which files to read, how to apply them, how to review. |
| `skills/codestd/core/conventions.md` | Principles that hold in every language. No numbers. |
| `skills/codestd/languages/<lang>.md` | Idioms, thresholds and linter settings for one language. |
| `.claude-plugin/` | Claude Code plugin and marketplace manifests. |

## Install

Claude Code, as a plugin:

```text
/plugin marketplace add tdnguyenND/codestd
/plugin install codestd@codestd
```

Claude Code, Codex, Cursor and other agents that read Agent Skills:

```bash
npx skills add tdnguyenND/codestd
```

## Use in a project

The skill loads when the agent judges a task relevant. To load it for every coding task, add this to
the project's `AGENTS.md` (or `CLAUDE.md`):

```markdown
## Coding conventions

Before writing, editing or reviewing code, load the `codestd` skill and follow it.
```

To review a change, ask for a codestd review, or run the skill with the `review` argument:
`/codestd:codestd review` when installed as a plugin, `/codestd review` when installed with
`npx skills`.

## Precedence

When rules conflict, the more specific one wins:

1. The project's own rules.
2. The language pack.
3. The core.

A language's official style guide, or an idiom a codebase has established, also overrides the core.

## Scope

Rules apply to code you add or change. Violations in code a change does not touch are reported, not
fixed in the same change.
